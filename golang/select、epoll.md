> 题目序号：427
>
> 题目来源：腾讯
>
> 整理人：lws (+)

**select**

**函数原型**

select系统调用时用来让我们的程序监视多个文件句柄的状态变化的。程序会停在select这里等待，直到被监视的文件句柄有一个或多个发生了状态改变。

```c
int select (int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout)

// 函数参数说明
nfds：最大文件描述符+1.所有加入集合的句柄值的最大那个那个值还要加1。因为句柄从0开始。
readfds：可读事件集合
writefds：可写事件集合
exceptfds：异常事件集合
timeout：用于设置select函数的超时时间，即告诉内核select等待多长时间之后就放弃等待。

// 函数返回值说明
1、做好准备的文件描述符的个数
2、返回0:超时;
3、返回-1：错误;
```

**总结**

select本质上是通过设置或者检查存放fd标志位的数据结构来进行下一步处理。

1. 单个进程可监视的fd数量被限制，即能监听端口的大小有限。一般来说这个数目和系统内存关系很大，具体数目可以cat/proc/sys/fs/file-max察看。32位机默认是1024个。64位机默认是2048.
1. 对socket进行扫描时是线性扫描，即采用轮询的方法，效率较低：当套接字比较多的时候，每次select()都要通过遍历FD_SETSIZE个Socket来完成调度,不管哪个Socket是活跃的,都遍历一遍。这会浪费很多CPU时间。如果能给套接字注册某个回调函数，当他们活跃时，自动完成相关操作，那就避免了轮询，这正是epoll与kqueue做的。
1. 需要维护一个用来存放大量fd的数据结构，这样会使得用户空间和内核空间在传递该结构时复制开销大。

**epoll**

**函数原型**

- **epoll_create**

函数原型 ：int epoll_create(int size);
功能说明 ：创建一个 epoll 对象，返回该对象的描述符，注意要使用 close 关闭该描述符。
参数说明 ：从 Linux 内核 2.6.8 版本起，size 这个参数就被忽略了，只要求 size 大于 0 即可。

- **epoll_ctl**

函数原型 ：int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
功能说明 ：操作控制 epoll 对象，主要涉及 epoll 红黑树上节点的一些操作，比如添加节点，删除节点，修改节点事件。
参数说明：

1. epfd：通过 epoll_create 创建的 epoll 对象句柄。
2. op：对红黑树的操作，添加节点、删除节点、修改节点监听的事件，分别对应 EPOLL_CTL_ADD，EPOLL_CTL_DEL，EPOLL_CTL_MOD。
   1. 添加事件：相当于往红黑树添加一个节点，每个客户端连接服务器后会有一个通讯套接字，每个连接的通讯套接字都不重复，所以这个通讯套接字就是红黑树的 key。
   1. 修改事件：把红黑树上监听的 socket 对应的监听事件做修改。
   1. 删除事件：相当于取消监听 socket 的事件。
3. fd：需要添加监听的 socket 描述符，可以是监听套接字，也可以是与客户端通讯的通讯套接字。event：事件信息。

4. epoll_wait

函数原型 ：int epoll_wait(int epid, struct epoll_event *events, int maxevents, int timeout);
功能说明 ：阻塞一段时间并等待事件发生，返回事件集合，也就是获取内核的事件通知。说白了就是遍历双向链表，把双向链表里的节点数据拷贝出来，拷贝完毕后就从双向链表移除。
参数说明：

1. epid：epoll_create 返回的 epoll 对象描述符。
1. events：存放就绪的事件集合，这个是传出参数。
1. maxevents：代表可以存放的事件个数，也就是 events 数组的大小。
1. timeout：阻塞等待的时间长短，以毫秒为单位，如果传入 -1 代表阻塞等待。

**实现机制**

设想一下如下场景：有100万个客户端同时与一个服务器进程保持着TCP连接。而每一时刻，通常只有几百上千个TCP连接是活跃的(事实上大部分场景都是这种情况)。如何实现这样的高并发？

在select/poll时代，服务器进程每次都把这100万个连接告诉操作系统(从用户态复制句柄数据结构到内核态)，让操作系统内核去查询这些套接字上是否有事件发生，轮询完后，再将句柄数据复制到用户态，让服务器应用程序轮询处理已发生的网络事件，这一过程资源消耗较大，因此，select/poll一般只能处理几千的并发连接。

epoll的设计和实现与select完全不同。epoll通过在Linux内核中申请一个简易的文件系统(文件系统一般用什么数据结构实现？B+树)。把原先的select/poll调用分成了3个部分：

1. 调用epoll_create()建立一个epoll对象(在epoll文件系统中为这个句柄对象分配资源)
1. 调用epoll_ctl向epoll对象中添加这100万个连接的套接字
1. 调用epoll_wait收集发生的事件的连接

如此一来，要实现上面说是的场景，只需要在进程启动时建立一个epoll对象，然后在需要的时候向这个epoll对象中添加或者删除连接。同时，epoll_wait的效率也非常高，因为调用epoll_wait时，并没有一股脑的向操作系统复制这100万个连接的句柄数据，内核也不需要去遍历全部的连接。

**触发方式**

1. **水平触发**

   水平触发的主要特点是，如果⽤户在监听epoll事件，当内核有事件的时候，会拷贝给用户态事件，但是如果用户只处理了⼀次，那么剩下没有处理的会在下⼀次epoll_wait再次返回该事件。

2. **边缘触发**

   边缘触发，相对跟水平触发相反，当内核有事件到达， 只会通知用户一次，至于用户处理还是不处理，
   以后将不会再通知。这样减少了拷贝过程，增加了性能，但是相对来说，如果用户忘记处理，将会产
   事件丢失的情况

**总结**

**区别**

1、支持一个进程所能打开的最大连接数

> select：单个进程所能打开的最大连接数有FD_SETSIZE宏定义，其大小是32个整数的大小（在32位的机器上，大小就是3232，同理64位机器上FD_SETSIZE为3264），当然我们可以对进行修改，然后重新编译内核，但是性能可能会受到影响，这需要进一步的测试。
> poll：poll本质上和select没有区别，但是它没有最大连接数的限制，原因是它是基于链表来存储的。
> epoll：虽然连接数有上限，但是很大，1G内存的机器上可以打开10万左右的连接，2G内存的机器可以打开20万左右的连接。

2、FD剧增后带来的IO效率问题

> select：因为每次调用时都会对连接进行线性遍历，所以随着FD的增加会造成遍历速度慢的“线性下降性能问题”。
> poll：同上
> epoll：因为epoll内核中实现是根据每个fd上的callback函数来实现的，只有活跃的socket才会主动调用callback，所以在活跃socket较少的情况下，使用epoll没有前面两者的线性下降的性能问题，但是所有socket都很活跃的情况下，可能会有性能问题。

3、 消息传递方式

> select：内核需要将消息传递到用户空间，都需要内核拷贝动作
> poll：同上
> epoll：epoll通过内核和用户空间共享一块内存来实现的。