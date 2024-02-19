### 概念

在一段程序中，每一个函数都会有自己的内存区域存放自己的局部变量、返回地址等，这些内存会由编译器在栈中进行分配，每一个函数都会分配一个栈桢，在函数运行结束后进行销毁，但是有些变量我们想在函数运行结束后仍然使用它，那么就需要把这个变量在堆上分配，这种从"栈"上逃逸到"堆"上的现象就成为内存逃逸。

在栈上分配的地址，一般由系统申请和释放，不会有额外性能的开销，比如函数的入参、局部变量、返回值等。在堆上分配的内存，如果要回收掉，需要进行 GC，那么GC 一定会带来额外的性能开销。编程语言不断优化GC算法，主要目的都是为了减少 GC带来的额外性能开销，变量一旦逃逸会导致性能开销变大。

### 逃逸机制

编译器会根据变量是否被外部引用来决定是否逃逸：

1. 如果函数外部没有引用，则优先放到栈中；
2. 如果函数外部存在引用，则必定放到堆中;
3. 如果栈上放不下，则必定放到堆上;

逃逸分析也就是由编译器决定哪些变量放在栈，哪些放在堆中，通过编译参数`-gcflag=-m`可以查看编译过程中的逃逸分析，发生逃逸的几种场景如下：

#### 指针逃逸

```
package main

func escape1() *int {
var a int = 1
return &a
}

func main() {
escape1()
}
```

通过`go build -gcflags=-m main.go`查看逃逸情况：

```
./main.go:4:6: moved to heap: a
```

函数返回值为局部变量的指针，函数虽然退出了，但是因为指针的存在，指向的内存不能随着函数结束而回收，因此只能分配在堆上。

#### 栈空间不足

```
package main

func escape2() {
s := make([]int, 0, 10000)
for index, _ := range s {
s[index] = index
}
}

func main() {
escape2()
}
```

通过`go build -gcflags=-m main.go`查看逃逸情况：

```
./main.go:4:11: make([]int, 10000, 10000) escapes to heap
```

当栈空间足够时，不会发生逃逸，但是当变量过大时，已经完全超过栈空间的大小时，将会发生逃逸到堆上分配内存。局部变量s占用内存过大，编译器会将其分配到堆上

#### 变量大小不确定

```
package main

func escape3() {
number := 10
s := make([]int, number) // 编译期间无法确定slice的长度
for i := 0; i < len(s); i++ {
s[i] = i
}
}

func main() {
escape3()
}
```

编译期间无法确定slice的长度，这种情况为了保证内存的安全，编译器也会触发逃逸，在堆上进行分配内存。直接`s := make([]int, 10)`不会发生逃逸

#### 动态类型

动态类型就是编译期间不确定参数的类型、参数的长度也不确定的情况下就会发生逃逸

空接口 interface{} 可以表示任意的类型，如果函数参数为 interface{}，编译期间很难确定其参数的具体类型，也会发生逃逸。

```
package main

import "fmt"

func escape4() {
fmt.Println(1111)
}

func main() {
escape4()
}
```

通过`go build -gcflags=-m main.go`查看逃逸情况：

```
./main.go:6:14: 1111 escapes to heap
```

fmt.Println(a ...interface{})函数参数为interface，编译器不确定参数的类型，会将变量分配到堆上

#### 闭包引用对象

```
package main

func escape5() func() int {
var i int = 1
return func() int {
i++
return i
}
}

func main() {
escape5()
}
```

通过`go build -gcflags=-m main.go`查看逃逸情况：

```
./main.go:4:6: moved to heap: i
```

闭包函数中局部变量i在后续函数是继续使用的，编译器将其分配到堆上

### 总结

1. 栈上分配内存比在堆中分配内存效率更高

2. 栈上分配的内存不需要 GC 处理，而堆需要

3. 逃逸分析目的是决定内分配地址是栈还是堆

4. 逃逸分析在编译阶段完成

因为无论变量的大小，只要是指针变量都会在堆上分配，所以对于小变量我们还是使用传值效率（而不是传指针）更高一点。