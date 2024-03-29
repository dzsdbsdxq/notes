> **题目来源**：滴滴 
>
> **频次**：1

答案：peace

线程是操作系统的内核对象，多线程编程时，如果线程数过多，就会导致频繁的上下文切换，这些 cpu 时间是一个额外的耗费。所以在一些高并发的网络服务器编程中，使用一个线程服务一个 socket 连接是很不明智的。于是操作系统提供了基于事件模式的异步编程模型。用少量的线程来服务大量的网络连接和I/O操作。但是采用异步和基于事件的编程模型，复杂化了程序代码的编写，非常容易出错。因为线程穿插，也提高排查错误的难度。

goroutine也是基于线程的。内部实现上，维护了一组数据结构和 n 个线程，真正的执行还是线程，goroutine执行的代码被扔进一个待执行队列中，由这 n 个线程从队列中拉出来执行。这就解决了goroutine的执行问题。那么goroutine是怎么切换的呢？答案是：golang 对各种 io函数进行了封装，这些封装的函数提供给应用程序使用，而其内部调用了操作系统的异步 io函数，当这些异步函数返回busy或bloking 时，golang 利用这个时机将现有的执行序列压栈，让线程去拉另外一个协程的代码来执行，基本原理就是这样，利用并封装了操作系统的异步函数。包括 linux 的 epoll、select 和 windows 的 iocp、event 等。