> 题目序号：439
>
> 题目来源：字节
>
> 整理人：lws

1. **make和new的区别**
   1. `new`函数主要是为类型申请一片内存空间，返回执行内存的指针
   2. `make`函数能够分配并初始化类型所需的内存空间和结构，返回复合类型的本身。
   3. `make`函数仅支持 `channel`、`map`、`slice` 三种类型，其他类型不可以使用使用`make`。
   4. `new`函数在日常开发中使用是比较少的，可以被替代。
   5. `make`函数初始化`slice`会初始化零值，日常开发要注意这个问题。

2. **`make`函数底层实现**

```go
func makeslice(et *_type, len, cap int) unsafe.Pointer {
	mem, overflow := math.MulUintptr(et.size, uintptr(cap))
	if overflow || mem > maxAlloc || len < 0 || len > cap {
		// NOTE: Produce a 'len out of range' error instead of a
		// 'cap out of range' error when someone does make([]T, bignumber).
		// 'cap out of range' is true too, but since the cap is only being
		// supplied implicitly, saying len is clearer.
		// See golang.org/issue/4085.
		mem, overflow := math.MulUintptr(et.size, uintptr(len))
		if overflow || mem > maxAlloc || len < 0 {
			panicmakeslicelen()
		}
		panicmakeslicecap()
	}

	return mallocgc(mem, et, true)
}
```

函数功能：

- 检查切片占用的内存空间是否溢出。
- 调用`mallocgc`在堆上申请一片连续的内存。

检查内存空间这里是根据切片容量进行计算的，根据当前切片元素的大小与切片容量的乘积得出当前内存空间的大小，检查溢出的条件：

- 内存空间大小溢出了
- 申请的内存空间大于最大可分配的内存
- 传入的`len`小于`0`，`cap`的大小只小于`len