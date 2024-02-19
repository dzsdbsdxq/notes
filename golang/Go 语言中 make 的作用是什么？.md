`make`的作用是为`slice, map or chan`的初始化 然后返回引用 `make`函数是内建函数，函数定义：

```go
func make(Type, size IntegerType) Type 
```

`make(T, args)`函数的目的和`new(T)`不同 仅仅用于创建`slice, map, channel` 而且返回类型是实例