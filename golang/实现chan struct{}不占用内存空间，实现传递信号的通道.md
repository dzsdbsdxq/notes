> 题目来源：BIGO

答案：树枝

~~~ go
// 空结构体的宽度是0，占用了0字节的内存空间。
// 所以空结构体组成的组合数据类型也不会占用内存空间。
channel := make(chan struct{})
go func() {
    // do something
    channel <- struct{}{}
}()
fmt.Println(<-channel)
~~~

