> 题目序号：（1490）
>
> 题目来源：字节跳动
>
> 频次：1

答案1：（jimyag）

```go
package main

import "fmt"

func main() {
	fooMap := make(map[string]string)
	fooMap["foo1Key"] = "foo1Value"
	fooMap["foo2Key"] = "foo2Value"
	fooMap["foo3Key"] = "foo3Value"
	fooMap["foo4Key"] = "foo4Value"
	fooMap["foo5Key"] = "foo5Value"
	fooMap["foo6Key"] = "foo6Value"

	for k, v := range fooMap {
		fmt.Printf("k: %s ,v: %s \n", k, v)
	}
	//k: foo1Key ,v: foo1Value
	//k: foo2Key ,v: foo2Value
	//k: foo3Key ,v: foo3Value
	//k: foo4Key ,v: foo4Value
	//k: foo5Key ,v: foo5Value
	//k: foo6Key ,v: foo6Value

	//k: foo5Key ,v: foo5Value
	//k: foo6Key ,v: foo6Value
	//k: foo1Key ,v: foo1Value
	//k: foo2Key ,v: foo2Value
	//k: foo3Key ,v: foo3Value
	//k: foo4Key ,v: foo4Value

}

```

结论：map遍历的时候每次的顺序是不一样的。

```go
go tool compile -S main.go 
...初始化map...
0x0253 00595 (main.go:14)       LEAQ    type.map[string]string(SB), AX
        0x025a 00602 (main.go:14)       LEAQ    ""..autotmp_13+104(SP), BX
        0x025f 00607 (main.go:14)       LEAQ    ""..autotmp_10+152(SP), CX
        0x0267 00615 (main.go:14)       PCDATA  $1, $2
        0x0267 00615 (main.go:14)       CALL    runtime.mapiterinit(SB)
        0x026c 00620 (main.go:14)       JMP     786
        0x0271 00625 (main.go:14)       MOVQ    ""..autotmp_10+160(SP), DX
        0x0279 00633 (main.go:14)       MOVQ    (DX), SI
        0x027c 00636 (main.go:14)       MOVQ    SI, "".v.ptr+64(SP)
        0x0281 00641 (main.go:14)       MOVQ    (CX), AX
        0x0284 00644 (main.go:14)       MOVQ    8(DX), DX
... 
```

在初始化完成之后，调用了`runtime.mapiterinit`方法，在`go1.18\src\runtime\map.go`中找到此方法的实现，看一下源码的实现。

```go
// mapiterinit initializes the hiter struct used for ranging over maps.
// The hiter struct pointed to by 'it' is allocated on the stack
// by the compilers order pass or on the heap by reflect_mapiterinit.
// Both need to have zeroed hiter since the struct contains pointers.
func mapiterinit(t *maptype, h *hmap, it *hiter) {
	if raceenabled && h != nil {
		callerpc := getcallerpc()
		racereadpc(unsafe.Pointer(h), callerpc, abi.FuncPCABIInternal(mapiterinit))
	}

	it.t = t
	if h == nil || h.count == 0 {
		return
	}

	if unsafe.Sizeof(hiter{})/goarch.PtrSize != 12 {
		throw("hash_iter size incorrect") // see cmd/compile/internal/reflectdata/reflect.go
	}
	it.h = h

	// grab snapshot of bucket state
	it.B = h.B
	it.buckets = h.buckets
	if t.bucket.ptrdata == 0 {
		// Allocate the current slice and remember pointers to both current and old.
		// This preserves all relevant overflow buckets alive even if
		// the table grows and/or overflow buckets are added to the table
		// while we are iterating.
		h.createOverflow()
		it.overflow = h.extra.overflow
		it.oldoverflow = h.extra.oldoverflow
	}

	// decide where to start
	r := uintptr(fastrand())
	if h.B > 31-bucketCntBits {
		r += uintptr(fastrand()) << 31
	}
	it.startBucket = r & bucketMask(h.B)
	it.offset = uint8(r >> h.B & (bucketCnt - 1))

	// iterator state
	it.bucket = it.startBucket

	// Remember we have an iterator.
	// Can run concurrently with another mapiterinit().
	if old := h.flags; old&(iterator|oldIterator) != iterator|oldIterator {
		atomic.Or8(&h.flags, iterator|oldIterator)
	}

	mapiternext(it)
}
```

通过对 `mapiterinit` 方法阅读，可得知其主要用途是在 map 进行遍历迭代时进行初始化动作。共有三个形参，用于读取当前哈希表的类型信息、当前哈希表的存储信息和当前遍历迭代的数据。

咱们关注到源码中 `fastrand` 的部分，这个方法名，是不是迷之眼熟。没错，它是一个生成随机数的方法。再看看上下文

```go
// decide where to start
	r := uintptr(fastrand())
	if h.B > 31-bucketCntBits {
		r += uintptr(fastrand()) << 31
	}
	it.startBucket = r & bucketMask(h.B)
	it.offset = uint8(r >> h.B & (bucketCnt - 1))
```

在这段代码中，它生成了随机数。用于决定从哪里开始循环迭代。更具体的话就是根据随机数，选择一个桶位置作为起始点进行遍历迭代

**因此每次重新 `for range map`，你见到的结果都是不一样的。那是因为它的起始位置根本就不固定！**

参考：[为什么遍历 Go map 是无序的？ - 云+社区 - 腾讯云 (tencent.com)](