> 题目来源：七牛

答案：T

参考《Go 语言底层原理剖析》

Go 语言的理念是通过通信来实现共享内存。Go 的CSP，通信顺序进程，是通过goroutine和channel来实现的。

<img src="https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/07f1b72a9bd56b8462377a1fc907f3bb.png" style="zoom:50%;" />

如上图所见:

通道在运行时是一个特殊的hchan结构体，

```go
type hchan struct{
	qcount uint            通道队列中的数据个数
	dataqsiz uint           通道队列中数据的大小
	buf unsafe.Pointer      存放实际数据的指针
	elemsize uint16         通道类型的大小
	closed uint32           通道是否关闭
	elemtype *_type         通道类型
	sendx uint              记录发送者在buf中的序号（记录往channel中写入数据的下一个位置）
	recvx uint              记录接收者在buf中的序号 （从channel中读取数据的下一个位置）
	recvq waitq             读取的阻塞协程队列  
	sendq waitq             写入的阻塞协程队列
	lock mutex              锁，并发保护
}
```

对于有缓冲的通道，存储在buf中的数据虽然是线性的数组，但是用数组和序号recvx，sendx模拟了一个环形队列。recvx可以找到从buf中哪个位置获取通道中的元素，而sendx能够找到写入时放到buf的位置，这样做主要是为了重用已经使用过的空间。recvx到sendx的距离代表通道队列中的元素数量。