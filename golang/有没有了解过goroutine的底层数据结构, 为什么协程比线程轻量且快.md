> 题目来源：小雨伞保险

答案：ORVR

**底层数据结构**

G是goroutine的缩写，相当于操作系统中的进程控制块，在这里就是goroutine的控制结构，是对goroutine的抽象。其中包括goid是这个goroutine的ID，status是这个goroutine的状态，如Gidle,Grunnable,Grunning,Gsyscall,Gwaiting,Gdead等。

```go
struct G{uintptr stackguard; // 分段栈的可用空间下界
uintptr stackbase; // 分段栈的栈基址
Gobuf sched; //进程切换时，利用sched域来保存上下文
uintptr stack0;FuncVal* fnstart; // goroutine运行的函数
void* param; // 用于传递参数，睡眠时其它
goroutine设置param，唤醒时此goroutine可以获取
int16 status; // 状态Gidle,Grunnable,Grunning,Gsyscall,Gwaiting,Gdead
int64 goid; // goroutine的id号G* schedlink;
M* m; // for debuggers, but offset not hard-coded
M* lockedm; // G被锁定只能在这个m上运行
uintptr gopc; // 创建这个goroutine的go表达式的pc...};
```

结构体G中的部分域如上所示。可以看到，其中包含了栈信息stackbase和stackguard，有运行的函数信息fnstart。这些就足够成为一个可执行的单元了，只要得到CPU就可以运行。goroutine切换时，上下文信息保存在结构体的sched域中。goroutine是轻量级的线程或者称为协程，切换时并不必陷入到操作系统内核中，所以保存过程很轻量。看一下结构体G中的Gobuf，其实只保存了当前栈指针，程序计数器，以及goroutine自身。

```go
struct Gobuf
{
    // The offsets of these fields are known to (hard-coded in) libmach.
    uintptr sp;
    byte* pc;
    G* g;
    ...
};
```

记录g是为了恢复当前goroutine的结构体G指针，运行时库中使用了一个常驻的寄存器extern register G* g，这个是当前goroutine的结构体G的指针。这样做是为了快速地访问goroutine中的信息，比如，Go的栈的实现并没有使用%ebp寄存器，不过这可以通过g->stackbase快速得到。”extern register”是由6c，8c等实现的一个特殊的存储。在ARM上它是实际的寄存器；其它平台是由段寄存器进行索引的线程本地存储的一个槽位。在linux系统中，对g和m使用的分别是0(GS)和4(GS)。需要注意的是，链接器还会根据特定操作系统改变编译器的输出，例如，6l/linux下会将0(GS)重写为-16(FS)。每个链接到Go程序的C文件都必须包含runtime.h头文件，这样C编译器知道避免使用专用的寄存器。

**为什么比线程快**

可以从三个角度区别：内存消耗、创建与销毀、切换。

**内存占用**

创建一个 goroutine 的栈内存消耗为 2 KB，实际运行过程中，如果栈空间不够用，会自动进行扩容。创建一个 thread 则需要消耗 1 MB 栈内存，而且还需要一个被称为 “a guard page” 的区域用于和其他 thread 的栈空间进行隔离。对于一个用 Go 构建的 HTTP Server 而言，对到来的每个请求，创建一个 goroutine 用来处理是非常轻松的一件事。而如果用一个使用线程作为并发原语的语言构建的服务，例如 Java 来说，每个请求对应一个线程则太浪费资源了，很快就会出 OOM 错误（OutOfMermoryError）。

**创建和销毀**

Thread 创建和销毀都会有巨大的消耗，因为要和操作系统打交道，是内核级的，通常解决的办法就是线程池。而 goroutine 因为是由 Go runtime 负责管理的，创建和销毁的消耗非常小，是用户级。

**切换**

当 threads 切换时，需要保存各种寄存器，以便将来恢复：16 general purpose registers, PC (Program Counter), SP (Stack Pointer), segment registers, 16 XMM registers, FP coprocessor state, 16 AVX registers, all MSRs etc.而 goroutines 切换只需保存三个寄存器：Program Counter, Stack Pointer and BP。一般而言，线程切换会消耗 1000-1500 纳秒，一个纳秒平均可以执行 12-18 条指令。所以由于线程切换，执行指令的条数会减少 12000-18000。Goroutine 的切换约为 200 ns，相当于 2400-3600 条指令。因此，goroutines 切换成本比 threads 要小得多。