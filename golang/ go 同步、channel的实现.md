> 题目序号：430
>
> 题目来源：腾讯
>
> 整理人：lws

1. **channel的基本概念**

channel俗称管道，用于数据传递或数据共享，其本质是一个先进先出的队列，使用goroutine+channel进行数据通讯简单高效，同时也线程安全，多个goroutine可同时修改一个channel，不需要加锁。

channel可分为三种类型：

1. 只读channel：只能读channel里面数据，不可写入
2. 只写channel：只能写数据，不可读
3. 一般channel：可读可写

4. **channel的数据结构**

```go
type hchan struct {
    qcount   uint // 队列中元素个数
    dataqsiz uint // 循环队列的大小
    buf      unsafe.Pointer // 指向循环队列
    elemsize uint16 // 通道里面的元素大小
    closed   uint32 // 通道关闭的标志
    elemtype *_type // 通道元素的类型
    sendx    uint   // 待发送的索引，即循环队列中的队尾指针front
    recvx    uint   // 待读取的索引，即循环队列中的队头指针rear
    recvq    waitq  // 接收等待队列
    sendq    waitq  // 发送等待队列
    lock mutex 		// 互斥锁
}
```

3. **channel的hchan结构图**

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/4ee100350cf645e88127ebee70d5ed73_tplv-k3u1fbpfcp-zoom-in-crop-mark_1304_0_0_0.awebp)

hchan结构体中的buf指向一个数组，用来实现循环队列，sendx是循环队列的队尾指针，recvx是循环队列的队头指针。dataqsize是缓存型通道的大小，qcount是记录通道内元素个数。

循环队列一般使用空余单元法来解决队空和队满时候都存在font=rear带来的二义性问题，但这样会浪费一个单元。golang的channel中是通过增加qcount字段记录队列长度来解决二义性，一方面不会浪费一个存储单元，另一方面当使用len函数查看队列长度时候，可以直接返回qcount字段，一举两得。

hchan结构体中另一重要部分是recvq,sendq，分别存储了等待从通道中接收数据的goroutine，和等待发送数据到通道的goroutine。两者都是waitq类型。

waitq是一个结构体类型，waitq和sudog构成双向链表，其中sudog是链表元素的类型，waitq中first和last字段分别指向链表头部的sudog，链表尾部的sudog。

