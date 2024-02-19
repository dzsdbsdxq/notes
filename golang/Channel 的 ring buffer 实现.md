channel 中使用了 ring buffer（环形缓冲区) 来缓存写入的数据。ring  buffer 有很多好处，而且非常适合用来实现 FIFO 式的固定长度队列。 在 channel 中，ring buffer 的实现如下：

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/_20220217140533.png)

上图展示的是一个缓冲区为 8 的 channel buffer，recvx 指向最早被读取的数 据，sendx 指向再次写入时插入的位置。