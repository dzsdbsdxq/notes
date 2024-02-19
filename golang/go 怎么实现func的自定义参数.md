> 题目来源：滴滴
>
> 频次：1

## 答案：树枝

在 golang中，type 可以定义任何自定义的类型

func 也是可以作为类型自定义的，type myFunc func(int) int，意思是自定义了一个叫 myFunc 的函数类型，这个函数的签名必须符合输入为 int，输出为 int。

golang通过type定义函数类型
通过 type 可以定义函数类型，格式如下

```go
type typeName func(arguments) retType
```

函数类型也是一种类型，故可以将其定义为函数入参，在 go 语言中函数名可以看做是函数类型的常量，所以我们可以直接将函数名作为参数传入的函数中。

~~~ go
package main

import "fmt"

func add(a, b int) int {
	return a + b
}

//sub作为函数名可以看成是 op 类型的常量
func sub(a, b int) int {
	return a - b
}

//定义函数类型 op
type op func(a, b int) int

//形参指定传入参数为函数类型op
func Oper(fu op, a, b int) int {
	return fu(a, b)
}

func main() {
	//在go语言中函数名可以看做是函数类型的常量，所以我们可以直接将函数名作为参数传入的函数中。
	aa := Oper(add, 1, 2)
	fmt.Println(aa)
	bb := Oper(sub, 1, 2)
	fmt.Println(bb)
}
~~~

#### 