> 题目序号：224
>
> 题目来源: 映客 
>
> 频次: 1

**答案1：**（苦痛律动）

*结构体* *指针方法* *值方法*

```go
type struct_variable_type struct {
    member definition
    member definition
    ...
    member definition
}

func (s *MyStruct) pointerMethod() { } // method on pointer
func (s MyStruct)  valueMethod()   { } // method on value
```

结构体Struct 与 结构体指针 区别当然是 一个是struct类型 一个是Pointer类型(*struct_variable_type)
但题目想问的应该是 *结构体值方法*调用 和 *结构体指针方法*调用 有什么区别?

```go
type MyStruct struct {
    Name string
}
func (s MyStruct) SetName1(name string) {
    s.Name = name
}
func (s *MyStruct) SetName2(name string) {
    s.Name = name
}
```

做一个测试

```go
s := MyStruct{Name: "test"}
fmt.Println(s.Name)
// pointer method
s.SetName1("test method")
fmt.Println(s.Name) // output test
// method
s.SetName2("test pointer method")
fmt.Println(s.Name) // output test pointer method
```

结论十分明显，结构体方法与结构体指针方法最大的区别是，只有指针方法才能修改接收器内(官方说法receiver)的值。(the receiver must be a pointer)。 结构体值方法调用的参数是结构体参数的拷贝(valueMethod is called with a copy of the caller's argument)。所以对原结构体无法进行修改
此外，如果结构体十分复杂，那么最好使用指针方法。（If the receiver is large, a big struct for instance, it will be much cheaper to use a pointer receiver.）
最后，需要保持一致性，如果该结构体有方法使用了指针方法，其余的方法也应该使用指针方法。（If some of the methods of the type must have pointer receivers, the rest should too, so the method set is consistent regardless of how the type is used.）

**总结**

1. 结构体值方法和结构体指针方法最大的区别是，指针方法可以修改接收器的值，值方法不行
2. 如果结构体十分复杂，效率考虑，使用指针方法更好，因为值方法需要对原接收器进行拷贝
3. 如果一个struct已经有了指针方法，一致性考虑，所有方法都应该统一使用指针方法

**参考资料**

1. https://segmentfault.com/a/1190000040129295
2. https://go.dev/doc/faq#methods_on_values_or_pointers