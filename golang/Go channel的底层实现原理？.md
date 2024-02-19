**概念：**

Go中的channel 是一个队列，遵循先进先出的原则，负责协程之间的通信（Go 语言提倡不要通过共享内存来通信，而要通过通信来实现内存共享，CSP(Communicating Sequential Process)并发模型，就是通过 goroutine 和 channel 来实现的）

**使用场景：**

停止信号监听

定时任务

生产方和消费方解耦

控制并发数

**底层数据结构：**

通过var声明或者make函数创建的channel变量是一个存储在函数栈帧上的指针，占用8个字节，指向堆上的hchan结构体

源码包中`src/runtime/chan.go`定义了hchan的数据结构：

![hchan](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/hchan.png)

hchan结构体：

```none
type hchan struct {
closed   uint32   // channel是否关闭的标志
elemtype *_type   // channel中的元素类型

// channel分为无缓冲和有缓冲两种。
// 对于有缓冲的channel存储数据，使用了 ring buffer（环形缓冲区) 来缓存写入的数据，本质是循环数组
// 为啥是循环数组？普通数组不行吗，普通数组容量固定更适合指定的空间，弹出元素时，普通数组需要全部都前移
// 当下标超过数组容量后会回到第一个位置，所以需要有两个字段记录当前读和写的下标位置
buf      unsafe.Pointer // 指向底层循环数组的指针（环形缓冲区）
qcount   uint           // 循环数组中的元素数量
dataqsiz uint           // 循环数组的长度
elemsize uint16                 // 元素的大小
sendx    uint           // 下一次写下标的位置
recvx    uint           // 下一次读下标的位置

// 尝试读取channel或向channel写入数据而被阻塞的goroutine
recvq    waitq  // 读等待队列
sendq    waitq  // 写等待队列

lock mutex //互斥锁，保证读写channel时不存在并发竞争问题
}
```

等待队列：

双向链表，包含一个头结点和一个尾结点

每个节点是一个sudog结构体变量，记录哪个协程在等待，等待的是哪个channel，等待发送/接收的数据在哪里

```
type waitq struct {
first *sudog
last  *sudog
}

type sudog struct {
g *g
next *sudog
prev *sudog
elem unsafe.Pointer 
c        *hchan 
...
}
```

**操作**：

**创建**

使用 `make(chan T, cap)` 来创建 channel，make 语法会在编译时，转换为 `makechan64` 和 `makechan`

```
func makechan64(t *chantype, size int64) *hchan {
if int64(int(size)) != size {
panic(plainError("makechan: size out of range"))
}

return makechan(t, int(size))
}
```

创建channel 有两种，一种是带缓冲的channel，一种是不带缓冲的channel

```
// 带缓冲
ch := make(chan int, 3)
// 不带缓冲
ch := make(chan int)
```

创建时会做一些检查:

- 元素大小不能超过 64K
- 元素的对齐大小不能超过 maxAlign 也就是 8 字节
- 计算出来的内存是否超过限制

创建时的策略:

- 如果是无缓冲的 channel，会直接给 hchan 分配内存
- 如果是有缓冲的 channel，并且元素不包含指针，那么会为 hchan 和底层数组分配一段连续的地址
- 如果是有缓冲的 channel，并且元素包含指针，那么会为 hchan 和底层数组分别分配地址

**发送**

发送操作，编译时转换为`runtime.chansend`函数

```
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool 
```

阻塞式：

调用chansend函数，并且block=true

```
ch <- 10
```

非阻塞式：

调用chansend函数，并且block=false

```
select {
case ch <- 10:
...

default
}
```

向 channel 中发送数据时大概分为两大块：检查和数据发送，数据发送流程如下：

- 如果 channel 的读等待队列存在接收者goroutine
- 将数据**直接发送**给第一个等待的 goroutine， **唤醒接收的 goroutine**
- 如果 channel 的读等待队列不存在接收者goroutine
- 如果循环数组buf未满，那么将会把数据发送到循环数组buf的队尾
- 如果循环数组buf已满，这个时候就会走阻塞发送的流程，将当前 goroutine 加入写等待队列，并**挂起等待唤醒**

**接收**

发送操作，编译时转换为`runtime.chanrecv`函数

```
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) 
```

阻塞式：

调用chanrecv函数，并且block=true

```
<ch

v := <ch

v, ok := <ch

// 当channel关闭时，for循环会自动退出，无需主动监测channel是否关闭，可以防止读取已经关闭的channel,造成读到数据为通道所存储的数据类型的零值
for i := range ch {
fmt.Println(i)
}
```

非阻塞式：

调用chanrecv函数，并且block=false

```
select {
case <-ch:
...

default
}
```

向 channel 中接收数据时大概分为两大块，检查和数据发送，而数据接收流程如下：

- 如果 channel 的写等待队列存在发送者goroutine
- 如果是无缓冲 channel，**直接**从第一个发送者goroutine那里把数据拷贝给接收变量，**唤醒发送的 goroutine**
- 如果是有缓冲 channel（已满），将循环数组buf的队首元素拷贝给接收变量，将第一个发送者goroutine的数据拷贝到 buf循环数组队尾，**唤醒发送的 goroutine**
- 如果 channel 的写等待队列不存在发送者goroutine
- 如果循环数组buf非空，将循环数组buf的队首元素拷贝给接收变量
- 如果循环数组buf为空，这个时候就会走阻塞接收的流程，将当前 goroutine 加入读等待队列，并**挂起等待唤醒**

**关闭**

关闭操作，调用close函数，编译时转换为`runtime.closechan`函数

```
close(ch)
```

```
func closechan(c *hchan) 
```

**案例分析：**

```
package main

import (
"fmt"
"time"
"unsafe"
)

func main() {
// ch是长度为4的带缓冲的channel
// 初始hchan结构体重的buf为空，sendx和recvx均为0
ch := make(chan string, 4)
fmt.Println(ch, unsafe.Sizeof(ch))
go sendTask(ch)
go receiveTask(ch)
time.Sleep(1 * time.Second)
}

// G1是发送者
// 当G1向ch里发送数据时，首先会对buf加锁，然后将task存储的数据copy到buf中，然后sendx++，然后释放对buf的锁
func sendTask(ch chan string) {
taskList := []string{"this", "is", "a", "demo"}
for _, task := range taskList {
ch <- task //发送任务到channel
}
}

// G2是接收者
// 当G2消费ch的时候，会首先对buf加锁，然后将buf中的数据copy到task变量对应的内存里，然后recvx++,并释放锁
func receiveTask(ch chan string) {
for {
task := <-ch                  //接收任务
fmt.Println("received", task) //处理任务
}
}
```

总结hchan结构体的主要组成部分有四个：

- 用来保存goroutine之间传递数据的循环数组：buf
- 用来记录此循环数组当前发送或接收数据的下标值：sendx和recvx
- 用于保存向该chan发送和从该chan接收数据被阻塞的goroutine队列： sendq 和 recvq
- 保证channel写入和读取数据时线程安全的锁：lock