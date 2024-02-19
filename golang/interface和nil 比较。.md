> 题目序号：6334
> 题目来源：畅天游
> 频次：0

解答：chris

```go
func main()  {
	var res1 []string
	if res1 == nil {
		fmt.Println("res1 is nil")
	}
	IsNil(res1)
	IsNil(nil)
}

func IsNil(i interface{}) {
	if i == nil {
		fmt.Println("i is nil")
		return
	}
	fmt.Println("i isn't nil")
}


// 运行结果
//res1 is nil
//i isn't nil
//i is nil
```

> 按照理解应该都输出`is nil`，但是为什么不一样呢？

- interface 不是单纯的值，而是分为类型和值。所以必须要类型和值同时都为 nil 的情况下，interface 的 nil 判断才会为 true。
- 一个 interface{} 类型的变量包含了 2 个指针，一个指针`指向值的类型`，另外一个指针`指向实际的值`。在 Go 源码中 runtime 包下，我们可以找到runtime.eface的定义。

```
type eface struct {
    _type *_type
    data  unsafe.Pointer
} 
type _type struct {
    size       uintptr // type size
    ptrdata    uintptr // size of memory prefix holding all pointers
    hash       uint32  // hash of type; avoids computation in hash tables
    tflag      tflag   // extra type information flags
    align      uint8   // alignment of variable with this type
    fieldalign uint8   // alignment of struct field with this type
    kind       uint8   // enumeration for C
    alg        *typeAlg  // algorithm table
    gcdata    *byte    // garbage collection data
    str       nameOff  // string form
    ptrToThis typeOff  // type for pointer to this type, may be zero
}
```

从中可以看到 interface 变量之所以可以接收任何类型变量，是因为其本质是一个对象，并记录其类型和数据块的指针。

- res1赋值给interface后，res1本质是值为nil，类型并不是nil，所以输出`i isn't nil`

- nil类型与值都为nil，所以输出 `i is nil`

- 如何避免？

  > 1）既然值为 nil 的具型变量赋值给空接口会出现如此莫名其妙的情况，我们不要这么做，赋值前先做判空处理，不为 nil 才赋给空接口； 
  >
  > 2）使用`reflect.ValueOf().IsNil()`来判断。不推荐这种做法，因为当空接口对应的具型是值类型，会 panic。