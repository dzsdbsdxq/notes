> 题目序号: 5720
>
> 题目来源: 知乎
>
> 频次: 1

答案：咸鱼没有早餐

虽然在 `src/builtin/builtin.go` 中可以找到 `string` 的定义

```go
type string string
```

但关于 `string` 更底层的定义则在 `src/runtime/string.go` 中

在 `rawstring` 的描述中写道，该函数为新的 string 分配内存空间，且返回的 stirng 与 []byte 都指向同一片空间，调用者应该使用 []byte 去填充 string 的内容

```go
func rawstring(size int) (s string, b []byte) {
	p := mallocgc(uintptr(size), nil, false)

	stringStructOf(&s).str = p
	stringStructOf(&s).len = size

	*(*slice)(unsafe.Pointer(&b)) = slice{p, size, size}

	return
}
```

在函数中 `p` 申请并指向了一块 `size` 大小的区域，同时作为返回值的 `b []byte` 切片中的指针也同样使用 `p`

同样在 `src/runtime/string.go` 中可以找到 `stringStructOf` 函数和对应的 `stringStruct` 结构体

```go
type stringStruct struct {
	str unsafe.Pointer
	len int
}

func stringStructOf(sp *string) *stringStruct {
	return (*stringStruct)(unsafe.Pointer(sp))
}
```

由此来看 `rawstring` 中的两条语句是为返回的 string 类型绑定了两个值，由此也可以认为 `stringStruct` 就是 string 的**原型**

<img src="https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/image-20220424153424275.png" alt="image-20220424153424275" style="zoom: 67%;" />

了解 string 的数据原型，然后来看具体的转换过程

```go
const tmpStringBufSize = 32

type tmpBuf [tmpStringBufSize]byte

func stringtoslicebyte(buf *tmpBuf, s string) []byte {
   var b []byte
   if buf != nil && len(s) <= len(buf) {
      *buf = tmpBuf{}
      b = buf[:len(s)]
   } else {
      b = rawbyteslice(len(s))
   }
   copy(b, s)
   return b
}
```

当 string 长度小于 32 时，直接使用一个缓冲数组 tmpBuf 接收字符串即可==if 中的两行，不懂==

当 string 长度大于 32 时，则要创建一个新的切片

```go
func rawbyteslice(size int) (b []byte) {
	cap := roundupsize(uintptr(size))
	p := mallocgc(cap, nil, false)
	if cap != uintptr(size) {
		memclrNoHeapPointers(add(p, uintptr(size)), cap-uintptr(size))
	}

	*(*slice)(unsafe.Pointer(&b)) = slice{p, size, int(cap)}
	return
}
```

最后对应 `copy()` 的具体执行，则在 `src/runtime/slice.go` 中 ==这部分源码看不懂了==

```go
func slicecopy(toPtr unsafe.Pointer, toLen int, fromPtr unsafe.Pointer, fromLen int, width uintptr) int {
	if fromLen == 0 || toLen == 0 {
		return 0
	}

	n := fromLen
	if toLen < n {
		n = toLen
	}

	if width == 0 {
		return n
	}

	size := uintptr(n) * width
	if raceenabled {
		callerpc := getcallerpc()
		pc := abi.FuncPCABIInternal(slicecopy)
		racereadrangepc(fromPtr, size, callerpc, pc)
		racewriterangepc(toPtr, size, callerpc, pc)
	}
	if msanenabled {
		msanread(fromPtr, size)
		msanwrite(toPtr, size)
	}
	if asanenabled {
		asanread(fromPtr, size)
		asanwrite(toPtr, size)
	}

	if size == 1 { // common case worth about 2x to do here
		// TODO: is this still worth it with new memmove impl?
		*(*byte)(toPtr) = *(*byte)(fromPtr) // known to be a byte pointer
	} else {
		memmove(toPtr, fromPtr, size)
	}
	return n
}
```

