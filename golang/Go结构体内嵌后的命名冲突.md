> 题目来源：腾讯

答案：千羽

```go
package main

import (
	"fmt"
)

type A struct {
	a int
}

type B struct {
	a int
}

type C struct {
	A
	B
}

func main() {
	c := &C{}
	c.A.a = 1
	fmt.Println(c)
}
// 输出 &{{1} {0}}
```

- 第7行和第11行分别定义了两个拥有a int字段的结构体。
- 第15行的结构体嵌入了A和B的结构体。
- 第21行实例化C结构体。
- 第22行按常规的方法，访问嵌入结构体A中的a字段，并赋值1。
- 第23行可以正常输出实例化C结构体。

接着，将第22行修改为如下代码：

```go
func main(){
	c:=&C{}
    c.a=1
    fmt.Println(c)
}
```

<img src="https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/20220505230948.png" style="zoom:50%;" />

此时再编译运行，编译器报错：

.main.go:22:3:ambiguousselectorc.a

编译器告知C的选择器a引起歧义，也就是说，编译器无法决定将1赋给C中的A还是B里的字段a。使用c.a引发二义性的问题一般应该由程序员逐级完整写出避免错误。

在使用内嵌结构体时，Go语言的编译器会非常智能地提醒我们可能发生的歧义和错误。

**解决：可以通过：c.A.a或者c.B.a 都可以正确得到对应的值**