一般来说，局部变量会在函数返回后被销毁，因此被返回的引用就成为了"无所指"的引用，程序会进入未知状态。

但这在 Go 中是安全的，Go 编译器将会对每个局部变量进行逃逸分析。如果发现局部变量的作用域超出该函数，则不会将内存分配在栈上，而是分配在堆上，因为他们不在栈区，即使释放函数，其内容也不会受影响。

```
package main

import "fmt"

func add(x, y int) *int {
res := 0
res = x + y
return &res
}

func main() {
fmt.Println(add(1, 2))
}
```

这个例子中，函数 `add` 局部变量 `res` 发生了逃逸。res作为返回值，在 main 函数中继续使用，因此 res 指向的内存不能够分配在栈上，随着函数结束而回收，只能分配在堆上。

编译时可以借助选项 `-gcflags=-m`，查看变量逃逸的情况

```
./main.go:6:2: res escapes to heap:
./main.go:6:2:   flow: ~r2 = &res:
./main.go:6:2:     from &res (address-of) at ./main.go:8:9
./main.go:6:2:     from return &res (return) at ./main.go:8:2
./main.go:6:2: moved to heap: res
./main.go:12:13: ... argument does not escape
0xc0000ae008
```

`res escapes to heap` 即表示 `res` 逃逸到堆上了。