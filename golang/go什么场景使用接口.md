> **题目序号：**252
>
> **题目来源：**映客
>
> **频次：**1

 **答案1：**（趁醉独饮痛）

**定义**

* Interface 是一个定义了方法签名的集合,用来指定对象的行为，如果对象做到了 Interface 中方法集定义的行为，那就可以说实现了 Interface；
* 这些方法可以在不同的地方被不同的对象实现，这些实现可以具有不同的行为；
* interface 的主要工作仅是提供方法名称签名,输入参数,返回类型。最终由具体的对象来实现方法，比如 struct；
* interface 初始化值为 nil；

```plain
使用 type 关键字来申明，interface 代表类型，大括号里面定义接口的方法集合
type Animal interface {
Bark() string
Walk() string
}
```

**注意**

* 只声明没赋值的interface 是nil interface，value和 type 都是 nil
* 只要赋值了，即使赋了一个值为nil类型，也不再是nil interface
  Go 允许不带任何方法的 interface ,这种类型的 interface 叫 **empty interface**。所有类型都实现了 empty interface,因为任何一种类型至少实现了 0 个方法。

典型的应用场景是 fmt包的Println方法，它能支持接收各种不同的类型的数据，并且输出到控制台,就是interface{}的功劳。

**判断 interface 变量存储的是哪种类型**

一个 interface 可被多种类型实现，有时候我们需要区分 interface 变量究竟存储哪种类型的值？类型断言提供对接口值的基础具体值的访问。

```go
//类型断言查看interface数据类型
t := i.(T)
//该语句断言接口值i保存的具体类型为T，并将T的基础值分配给变量t。
//如果i保存的值不是类型 T ，将会触发 panic 错误。
```
