**“不要通过共享内存来通信，我们应该使用通信来共享内存”** 这句话想必大家已经非常熟悉了，在官方的博客，初学时的教程，甚至是在 Go 的源码中都能看到

无论是通过共享内存来通信还是通过通信来共享内存，最终我们应用程序都是读取的内存当中的数据，只是前者是直接读取内存的数据，而后者是通过发送消息的方式来进行同步。而通过发送消息来同步的这种方式常见的就是 Go 采用的 CSP(Communication Sequential Process) 模型以及 Erlang 采用的 Actor 模型，这两种方式都是通过通信来共享内存。

![02_Go进阶03_blog_channel.png](https://img.lailin.xyz/image/1610460699237-f6400aaa-34d5-4c8d-b323-27683704abd2.png)

大部分的语言采用的都是第一种方式直接去操作内存，然后通过互斥锁，CAS 等操作来保证并发安全。Go 引入了 Channel 和 Goroutine 实现 CSP 模型将生产者和消费者进行了解耦，Channel 其实和消息队列很相似。而 Actor 模型和 CSP 模型都是通过发送消息来共享内存，但是它们之间最大的区别就是 Actor 模型当中并没有一个独立的 Channel 组件，而是 Actor 与 Actor 之间直接进行消息的发送与接收，每个 Actor 都有一个本地的“信箱”消息都会先发送到这个“信箱当中”。

**优点**

- 使用 channel 可以帮助我们解耦生产者和消费者，可以降低并发当中的耦合

**缺点**

- 容易出现死锁的情况