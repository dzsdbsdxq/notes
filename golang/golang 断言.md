> 题目序号：6703
> 题目来源：腾讯
> 频次：4

答案：苏木

golang 断言是作用在接口上的。
go 作为一门强类型语言，对数据类型有着严格的区分，但所有类型（如 int、slice、map 等）都满足了 interface{} 接口，因为 interface{} 是没有方法的接口，也叫空接口。同时 interface{} 也是一种类型。

go 的接口值由`一个具体类型`和`具体类型的值`两部分组成，这两部分分别称为接口的`动态类型`和`动态值`。

对于非空接口：
非空接口的底层数据结构是 iface，其代码如下（代码位于src/runtime/runtime2.go中）：

```go
type iface struct{
    tab *itab			 	//itab存放类型及方法指针信息
    data unsafe.Pointer 	//数据信息
}
```

* itab：指针类型，用来存放接口自身类型和绑定的实例类型及实例相关的函数指针。
* 数据指针data：指向接口绑定的实例的副本，接口的初始化也是一种值拷贝。

对于空接口：
空接口的底层数据结构是 eface ，其代码如下：

```go
type eface struct{
    _type *_type			//_type存放类型
    data unsafe.Pointer 	//数据信息
}
```

从 eface 的数据结构可以看出，空接口并不是真的为空，其保留了具体实例的类型和值拷贝。

**类型断言**

当我们尝试封装一个方法时，有时候返回的是interface{}类型。若此时需要找到该 interface{} 类型的真实类型，就需要对其进行断言操作。

我们可以通过一种方法来获取接口的底层数据结构内的真实类型 —— 类型断言，其语法格式为：

```go
x.(T)   // x 是 interface{} 类型
        // T 是断言 x 可能的类型
```

示例代码：

```go
type Student struct {
    // nill
}
```

```go
package main

import "fmt"

func main() {
    
    var i1 interface{} = new (Student)
    s := i1.(Student) // 不安全，如果断言失败，会直接 panic

    fmt.Println(s)
}
```

同时，该语法返回两个参数，第一个参数是x转化为T类型后的变量，第二个值是一个布尔值，若为true则表示断言成功，为false则表示断言失败。

```go
func main() {
    
    var i2 interface{} = new (Student)
    s, ok := i2.(Student) // 安全，断言失败也不会 panic
    if ok {
        fmt.Println(s)
    }
}
```

除此之外，断言其实还有另一种形式，就是利用 type_switch 语句判断接口的类型。每个 case 会被顺序地考虑到，因此 case 语句地顺序是很重要的，同时也有可能会有多个 case 能够匹配。

```go
switch ins := s.(type) {
    case Triangle:
        fmt.Println("三角形...", ins.a, ins.b, ins.c)
    case Circle:
        fmt.Println("圆形...", ins.radius)
    case int:
        fmt.Println("整型数据...")
}
```

>事实上，使用 interface 类型进行转换也会对应用程序的性能产生影响。
>https://blog.csdn.net/flynetcn/article/details/119921587

**参考资料**

Go语言类型断言 https://blog.csdn.net/bestzy6/article/details/116243421

https://blog.csdn.net/weixin_36338224/article/details/106112067

https://blog.csdn.net/qq_49723651/article/details/121612693

https://lrbbfc.blog.csdn.net/article/details/110196299

断言对性能的影响 https://blog.csdn.net/flynetcn/article/details/119921587

GOLAND ROAD MAP 站内资源导航 https://www.golangroadmap.com/class/goinitial/11-2.html#_1-1-接口的类型