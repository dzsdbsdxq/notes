> 题目序号：（1509）
>
> 题目来源：腾讯
>
> 频次：1

答案1：（jimyag）

Go中的所有程序都实现了`interface{}`的接口，这意味着，所有的类型如`string`,`int`,`int64`甚至是自定义的`struct`类型都就此拥有了`interface{}`的接口.那么在一个数据通过`func funcName(interface{})`的方式传进来的时候，也就意味着这个参数被自动的转为interface{}的类型。

如以下的代码：

```go
func funcName(a interface{}) string {     return string(a)}
```

编译器将会返回：`cannot convert a (type interface{}) to type string: need type assertion`

此时，意味着整个转化的过程需要类型断言。类型断言有以下几种形式：

1. 直接断言使用

```go
var a interface{}fmt.Println("Where are you,Jonny?", a.(string))
```

但是如果断言失败一般会导致panic的发生。所以为了防止panic的发生，我们需要在断言前进行一定的判断

```go
value, ok := a.(string)
```

如果断言失败，那么ok的值将会是false,但是如果断言成功ok的值将会是true,同时value将会得到所期待的正确的值。示例：

```go
value, ok := a.(string)
if !ok {
    fmt.Println("It's not ok for type string")
    return
}
fmt.Println("The value is ", value)
```


另外也可以配合switch语句进行判断：

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T", t)       // %T prints whatever type t has    break
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool    break
case int:
    fmt.Printf("integer %d\n", t)             // t has type int    break
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool    break
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int    break
}
```
