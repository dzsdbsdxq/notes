Go 语言中，不要通过共享内存来通信，而要通过通信来实现内存共享。Go 的 CSP(Communicating Sequential Process)并发模型，中文可以叫做通信顺序进 程，是通过 goroutine 和 channel 来实现的。

channel 收发遵循先进先出 FIFO 的原则。分为有缓冲区和无缓冲区，channel 中包括 buffer、sendx 和 recvx 收发的位置(ring buffer 记录实现)、sendq、 recv。当 channel 因为缓冲区不足而阻塞了队列，则使用双向链表存储。