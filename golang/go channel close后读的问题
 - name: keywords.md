> 题目来源：UCLOUD
>
> 频次：高频

## 答案：树枝

channel作为传递消息的通道，对他的操作无非有三种，向channel发送值、从channel中取值，关闭channel。

对一个已经关闭的channel取值，如果里面有值会先取值，当值完后会取到该channel类型的零值

向一个已近关闭的channel发送至，会panic: send on closed channel

关闭一个已经关闭的channel，会panic: close of closed channel