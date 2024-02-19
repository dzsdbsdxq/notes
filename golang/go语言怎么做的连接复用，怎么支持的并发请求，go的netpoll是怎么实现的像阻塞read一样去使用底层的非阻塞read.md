> **题目序号：**（56）
>
> **题目来源：**字节跳动
>
> **频次：**1

**答案1：**（栾龙生）

[IO多路复用的netpoll模型](https://www.cnblogs.com/luozhiyun/p/14390824.html)

1. **go语言怎么做的连接复用**

   go语言中IO多路复用使用netpool模型

   netpoll本质上是对 I/O 多路复用技术的封装，所以自然也是和epoll一样脱离不了下面几步：

   1. netpoll创建及其初始化；
   2. 向netpoll中加入待监控的任务；
   3. 从netpoll获取触发的事件；

   在go中对epoll提供的三个函数进行了封装

   ```go
   func netpollinit()
   func netpollopen(fd uintptr, pd *pollDesc) int32
   func netpoll(delay int64) gList
   ```

   netpollinit函数负责初始化netpoll；

   netpollopen负责监听文件描述符上的事件；

   netpoll会阻塞等待返回一组已经准备就绪的 Goroutine；

2. **go语言怎么支持的并发请求**

   Go中有goroutine，所以可以采用多协程来解决并发问题。accept连接后，将连接丢给goroutine处理后续的读写操作。在开发者看到的这个goroutine中业务逻辑是同步的，也不用考虑IO是否阻塞。