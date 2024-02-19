> **题目序号：**(268)
> **题目来源：** 大疆
> **频次:** 1   

## 答案：阿纪、

1. **interface介绍**
   interface 是GO语言的基础特性之一。
   可以理解为一种类型的规范或者约定。
   它跟java，C# 不太一样，不需要显示说明实现了某个接口，它没有继承或子类或“implements”关键字，只是通过约定的形式，隐式的实现interface 中的方法即可。因此，Golang 中的 interface 让编码更灵活、易扩展。

2. **如何理解go 语言中的interface ?**

   1. interface 可以被任意对象实现，一个类型/对象也可以实现多个 interface
   2. 方法不能重载，如eat(), eat(s string)不能同时存在

3. **代码示例**

```go
type Phone interface {
   call()
}

type HuaWeiPhone struct {}

func (huaWeiPhone HuaWeiPhone) call() {
    fmt.Println("I am HuaWei, I can call you!")
}

type ApplePhone struct {}

func (iPhone ApplePhone) call() {
    fmt.Println("I am Apple Phone, I can call you!")
}

func main() {
    var phone Phone
    phone = new(HuaWeiPhone)
    phone.call()

    phone = new(ApplePhone)
    phone.call()
}
```

