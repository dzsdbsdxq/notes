> **题目序号：**655
>
> **题目来源**：网易
>
> **频次**：1

**答案1：**（重拾）

1.channel的发送与接收，从channel发送数据和读取数据需要使用  “<-”符号，如下图所示,

```go
//表示val值 将发到channel中
channel <- val 

//表示从channel中读取一个值并赋值到val上
val := <-channel

//表示需要检查ok是否为true，用于判断是否读取到了有效数据
val,ok := <-channel
```

2.带缓冲区的channel

创建channel时需要借助make函数对channel进行初始化

```go
ch :=make(chan T ,sizeOfChan)
```

在创建channel时需要指定channel传输的数据类型，可以选择指定channel的长度。如不指定，那么往channel中发送数据的goroutine将会阻塞，直到数据被读取；而指定了长度的channel，在缓冲区未满时发送数据不会被阻塞。