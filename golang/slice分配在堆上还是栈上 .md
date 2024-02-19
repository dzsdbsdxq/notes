> 题目序号：4562
>
> 题目来源：京东
>
> 频次：3

## 答案：栾龙生

有可能分配到栈上，也有可能分配到栈上。当开辟切片空间较大时，会逃逸到堆上。

通过命令`go build -gcflags "-m -l" xxx.go`观察golang是如何进行逃逸分析的

```go
package main

func main() {
	_ = make([]string, 200)		 	//1
    //_ = make([]string, 20000)		//2
}

//output
//1. make([]string, 200) does not escape
//2. make([]string, 20000) escapes to heap
```

