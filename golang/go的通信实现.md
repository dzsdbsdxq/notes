> 题目来源：滴滴

## 答案：flare

Go通信是通过channel实现的，chan定义实现了环形队列，总是遵循先入先出（First In First Out）的规则，保证收发数据的顺序，这一点和管道是一样的；chan在实现时定义了:

* 指针

* 环形队列

* （阻塞)协程链表

来控制通信，当chan满足条件时，通过指针sendx 、recvx 进行读写数据