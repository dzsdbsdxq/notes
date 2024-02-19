> 题目来源：字节跳动

答案：小禾先生

**设计原理**

今天的 Go 语言调度器有着优异的性能，但是如果我们回头看 Go 语言的 0.x 版本的调度器会发现最初的调度器不仅实现非常简陋，也无法支撑高并发的服务。调度器经过几个大版本的迭代才有今天的优异性能，历史上几个不同版本的调度器引入了不同的改进，也存在着不同的缺陷:

- 单线程调度器 · 0.x
  - 只包含 40 多行代码；
  - 程序中只能存在一个活跃线程，由 G-M 模型组成；
- 多线程调度器 · 1.0
  - 允许运行多线程的程序；
  - 全局锁导致竞争严重；
- 任务窃取调度器 · 1.1
  - 引入了处理器 P，构成了目前的 **G-M-P** 模型；
  - 在处理器 P 的基础上实现了基于**工作窃取**的调度器；
  - 在某些情况下，Goroutine 不会让出线程，进而造成饥饿问题；
  - 时间过长的垃圾回收（Stop-the-world，STW）会导致程序长时间无法工作；
- 抢占式调度器 · 1.2 ~ 至今
  - 基于协作的抢占式调度器 - 1.2 ~ 1.13
    - 通过编译器在函数调用时插入**抢占检查**指令，在函数调用时检查当前 Goroutine 是否发起了抢占请求，实现基于协作的抢占式调度；
    - Goroutine 可能会因为垃圾回收和循环长时间占用资源导致程序暂停；
  - 基于信号的抢占式调度器 - 1.14 ~ 至今
    - 实现**基于信号的真抢占式调度**；
    - 垃圾回收在扫描栈时会触发抢占调度；
    - 抢占的时间点不够多，还不能覆盖全部的边缘情况；
- 非均匀存储访问调度器 · 提案
  - 对运行时的各种资源进行分区；
  - 实现非常复杂，到今天还没有提上日程；

除了多线程、任务窃取和抢占式调度器之外，Go 语言社区目前还有一个非均匀存储访问（Non-uniform memory access，NUMA）调度器的提案。在这一节中，我们将依次介绍不同版本调度器的实现原理以及未来可能会实现的调度器提案。

**单线程调度器**

0.x 版本调度器只包含表示 Goroutine 的 G 和表示线程的 M 两种结构，全局也只有一个线程。我们可以在 [clean up scheduler](https://github.com/golang/go/commit/96824000ed89d13665f6f24ddc10b3bf812e7f47) 提交中找到单线程调度器的源代码，在这时 Go 语言的调度器还是由 C 语言实现的，调度函数 [`runtime.scheduler:9682400`](https://draveness.me/golang/tree/runtime.scheduler:9682400) 也只包含 40 多行代码 ：

```c
static void scheduler(void) {
	G* gp;
	lock(&sched);

	if(gosave(&m->sched)){
		lock(&sched);
		gp = m->curg;
		switch(gp->status){
		case Grunnable:
		case Grunning:
			gp->status = Grunnable;
			gput(gp);
			break;
		...
		}
		notewakeup(&gp->stopped);
	}

	gp = nextgandunlock();
	noteclear(&gp->stopped);
	gp->status = Grunning;
	m->curg = gp;
	g = gp;
	gogo(&gp->sched);
}
```

该函数会遵循如下的过程调度 Goroutine：

1. 获取调度器的全局锁；
2. 调用 [`runtime.gosave:9682400`](https://draveness.me/golang/tree/runtime.gosave:9682400) 保存栈寄存器和程序计数器；
3. 调用 [`runtime.nextgandunlock:9682400`](https://draveness.me/golang/tree/runtime.nextgandunlock:9682400) 获取下一个需要运行的 Goroutine 并解锁调度器；
4. 修改全局线程 `m` 上要执行的 Goroutine；
5. 调用 [`runtime.gogo:9682400`](https://draveness.me/golang/tree/runtime.gogo:9682400) 函数运行最新的 Goroutine；

虽然这个单线程调度器的唯一优点就是**能运行**，但是这次提交已经包含了 G 和 M 两个重要的数据结构，也建立了 Go 语言调度器的框架。

**多线程调度器**

Go 语言在 1.0 版本正式发布时就支持了多线程的调度器，与上一个版本几乎不可用的调度器相比，Go 语言团队在这一阶段实现了从不可用到可用的跨越。我们可以在 [`pkg/runtime/proc.c`](https://github.com/golang/go/blob/go1.0.1/src/pkg/runtime/proc.c) 文件中找到 1.0.1 版本的调度器，多线程版本的调度函数 [`runtime.schedule:go1.0.1`](https://draveness.me/golang/tree/runtime.schedule:go1.0.1) 包含 70 多行代码，我们在这里保留了该函数的核心逻辑：

```c
static void schedule(G *gp) {
	schedlock();
	if(gp != nil) {
		gp->m = nil;
		uint32 v = runtime·xadd(&runtime·sched.atomic, -1<<mcpuShift);
		if(atomic_mcpu(v) > maxgomaxprocs)
			runtime·throw("negative mcpu in scheduler");

		switch(gp->status){
		case Grunning:
			gp->status = Grunnable;
			gput(gp);
			break;
		case ...:
		}
	} else {
		...
	}
	gp = nextgandunlock();
	gp->status = Grunning;
	m->curg = gp;
	gp->m = m;
	runtime·gogo(&gp->sched, 0);
}
```

整体的逻辑与单线程调度器没有太多区别，因为我们的程序中可能同时存在多个活跃线程，所以多线程调度器引入了 `GOMAXPROCS` 变量帮助我们灵活控制程序中的最大处理器数，即活跃线程数。

多线程调度器的主要问题是调度时的锁竞争会严重浪费资源，[Scalable Go Scheduler Design Doc](http://golang.org/s/go11sched) 中对调度器做的性能测试发现 14% 的时间都花费在 [`runtime.futex:go1.0.1`](https://draveness.me/golang/tree/runtime.futex:go1.0.1) 上，该调度器有以下问题需要解决：

1. 调度器和锁是全局资源，所有的调度状态都是中心化存储的，锁竞争问题严重；
2. 线程需要经常互相传递可运行的 Goroutine，引入了大量的延迟；
3. 每个线程都需要处理内存缓存，导致大量的内存占用并影响数据局部性；
4. 系统调用频繁阻塞和解除阻塞正在运行的线程，增加了额外开销；

这里的全局锁问题和 Linux 操作系统调度器在早期遇到的问题比较相似，解决的方案也都大同小异。

**任务窃取调度器**

2012 年 Google 的工程师 Dmitry Vyukov 在 [Scalable Go Scheduler Design Doc](http://golang.org/s/go11sched) 中指出了现有多线程调度器的问题并在多线程调度器上提出了两个改进的手段：

1. 在当前的 G-M 模型中引入了处理器 P，增加中间层；
2. 在处理器 P 的基础上实现基于工作窃取的调度器；

基于任务窃取的 Go 语言调度器使用了沿用至今的 G-M-P 模型，我们能在 [runtime: improved scheduler](https://github.com/golang/go/commit/779c45a50700bda0f6ec98429720802e6c1624e8) 提交中找到任务窃取调度器刚被实现时的[源代码](https://github.com/golang/go/blob/779c45a50700bda0f6ec98429720802e6c1624e8/src/pkg/runtime/proc.c)，调度器的 [`runtime.schedule:779c45a`](https://draveness.me/golang/tree/runtime.schedule:779c45a) 在这个版本的调度器中反而更简单了：

```go
static void schedule(void) {
    G *gp;
 top:
    if(runtime·gcwaiting) {
        gcstopm();
        goto top;
    }

    gp = runqget(m->p);
    if(gp == nil)
        gp = findrunnable();

    ...

    execute(gp);
}
```

1. 如果当前运行时在等待垃圾回收，调用 [`runtime.gcstopm:779c45a`](https://draveness.me/golang/tree/runtime.gcstopm:779c45a) 函数；
2. 调用 [`runtime.runqget:779c45a`](https://draveness.me/golang/tree/runtime.runqget:779c45a) 和 [`runtime.findrunnable:779c45a`](https://draveness.me/golang/tree/runtime.findrunnable:779c45a) 从本地或者全局的运行队列中获取待执行的 Goroutine；
3. 调用 [`runtime.execute:779c45a`](https://draveness.me/golang/tree/runtime.execute:779c45a) 在当前线程 M 上运行 Goroutine；

当前处理器本地的运行队列中不包含 Goroutine 时，调用 [`runtime.findrunnable:779c45a`](https://draveness.me/golang/tree/runtime.findrunnable:779c45a) 会触发工作窃取，从其它的处理器的队列中随机获取一些 Goroutine。

运行时 G-M-P 模型中引入的处理器 P 是线程和 Goroutine 的中间层，我们从它的结构体中就能看到处理器与 M 和 G 的关系：

```c
struct P {
	Lock;

	uint32	status;
	P*	link;
	uint32	tick;
	M*	m;
	MCache*	mcache;

	G**	runq;
	int32	runqhead;
	int32	runqtail;
	int32	runqsize;

	G*	gfree;
	int32	gfreecnt;
};
```

处理器持有一个由可运行的 Goroutine 组成的环形的运行队列 `runq`，还反向持有一个线程。调度器在调度时会从处理器的队列中选择队列头的 Goroutine 放到线程 M 上执行。如下所示的图片展示了 Go 语言中的线程 M、处理器 P 和 Goroutine 的关系。

![golang-gmp](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/2020-02-02-15805792666151-golang-gmp.png)

基于工作窃取的多线程调度器将每一个线程绑定到了独立的 CPU 上，这些线程会被不同处理器管理，不同的处理器通过工作窃取对任务进行再分配实现任务的平衡，也能提升调度器和 Go 语言程序的整体性能，今天所有的 Go 语言服务都受益于这一改动。

**抢占式调度器**

对 Go 语言并发模型的修改提升了调度器的性能，但是 1.1 版本中的调度器仍然不支持抢占式调度，程序只能依靠 Goroutine 主动让出 CPU 资源才能触发调度。Go 语言的调度器在 1.2 版本[4](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/#fn:4)中引入基于协作的抢占式调度解决下面的问题[5](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/#fn:5)：

- 某些 Goroutine 可以长时间占用线程，造成其它 Goroutine 的饥饿；
- 垃圾回收需要暂停整个程序（Stop-the-world，STW），最长可能需要几分钟的时间[6](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/#fn:6)，导致整个程序无法工作；

1.2 版本的抢占式调度虽然能够缓解这个问题，但是它实现的抢占式调度是基于协作的，在之后很长的一段时间里 Go 语言的调度器都有一些无法被抢占的边缘情况，例如：for 循环或者垃圾回收长时间占用线程，这些问题中的一部分直到 1.14 才被基于信号的抢占式调度解决。

**基于协作的抢占式调度**

我们可以在 [`pkg/runtime/proc.c`](https://github.com/golang/go/blob/go1.2/src/pkg/runtime/proc.c) 文件中找到引入基于协作的抢占式调度后的调度器。Go 语言会在分段栈的机制上实现抢占调度，利用编译器在分段栈上插入的函数，所有 Goroutine 在函数调用时都有机会进入运行时检查是否需要执行抢占。Go 团队通过以下的多个提交实现该特性：

- runtime: add stackguard0 to G
  - 为 Goroutine 引入 `stackguard0` 字段，该字段被设置成 `StackPreempt` 意味着当前 Goroutine 发出了抢占请求；
- runtime: introduce preemption function (not used for now)
  - 引入抢占函数 [`runtime.preemptone:1e112cd`](https://draveness.me/golang/tree/runtime.preemptone:1e112cd) 和 [`runtime.preemptall:1e112cd`](https://draveness.me/golang/tree/runtime.preemptall:1e112cd)，这两个函数会改变 Goroutine 的 `stackguard0` 字段发出抢占请求；
  - 定义抢占请求 `StackPreempt`；
- runtime: preempt goroutines for GC
  - 在 [`runtime.stoptheworld:1e112cd`](https://draveness.me/golang/tree/runtime.stoptheworld:1e112cd) 中调用 [`runtime.preemptall:1e112cd`](https://draveness.me/golang/tree/runtime.preemptall:1e112cd) 设置所有处理器上正在运行的 Goroutine 的 `stackguard0` 为 `StackPreempt`；
  - 在 [`runtime.newstack:1e112cd`](https://draveness.me/golang/tree/runtime.newstack:1e112cd) 中增加抢占的代码，当 `stackguard0` 等于 `StackPreempt` 时触发调度器抢占让出线程；
- runtime: preempt long-running goroutines
  - 在系统监控中，如果一个 Goroutine 的运行时间超过 10ms，就会调用 [`runtime.retake:1e112cd`](https://draveness.me/golang/tree/runtime.retake:1e112cd) 和 [`runtime.preemptone:1e112cd`](https://draveness.me/golang/tree/runtime.preemptone:1e112cd)；
- runtime: more reliable preemption
  - 修复 Goroutine 因为周期性执行非阻塞的 CGO 或者系统调用不会被抢占的问题；

上面的多个提交实现了抢占式调度，但是还缺少最关键的一个环节 — 编译器如何在函数调用前插入函数，我们能在非常古老的提交 [runtime: stack growth adjustments, cleanup](https://github.com/golang/go/commit/7343e03c433ebb0c302ed97bf832ad3bd3170de6) 中找到编译器插入函数的雏形，最新版本的 Go 语言会通过 [`cmd/internal/obj/x86.stacksplit`](https://draveness.me/golang/tree/cmd/internal/obj/x86.stacksplit) 插入 [`runtime.morestack`](https://draveness.me/golang/tree/runtime.morestack)，该函数可能会调用 [`runtime.newstack`](https://draveness.me/golang/tree/runtime.newstack) 触发抢占。从上面的多个提交中，我们能归纳出基于协作的抢占式调度的工作原理：

1. 编译器会在调用函数前插入 [`runtime.morestack`](https://draveness.me/golang/tree/runtime.morestack)；
2. Go 语言运行时会在垃圾回收暂停程序、系统监控发现 Goroutine 运行超过 10ms 时发出抢占请求 `StackPreempt`；
3. 当发生函数调用时，可能会执行编译器插入的 [`runtime.morestack`](https://draveness.me/golang/tree/runtime.morestack)，它调用的 [`runtime.newstack`](https://draveness.me/golang/tree/runtime.newstack) 会检查 Goroutine 的 `stackguard0` 字段是否为 `StackPreempt`；
4. 如果 `stackguard0` 是 `StackPreempt`，就会触发抢占让出当前线程；

这种实现方式虽然增加了运行时的复杂度，但是实现相对简单，也没有带来过多的额外开销，总体来看还是比较成功的实现，也在 Go 语言中使用了 10 几个版本。因为这里的抢占是通过编译器插入函数实现的，还是需要函数调用作为入口才能触发抢占，所以这是一种**协作式的抢占式调度**。

**基于信号的抢占式调度**

基于协作的抢占式调度虽然实现巧妙，但是并不完备，我们能在 [runtime: non-cooperative goroutine preemption](https://github.com/golang/go/issues/24543) 中找到一些遗留问题：

- [runtime: tight loops should be preemptible #10958](https://github.com/golang/go/issues/10958)
- [An empty for{} will block large slice allocation in another goroutine, even with GOMAXPROCS > 1 ? #17174](https://github.com/golang/go/issues/17174)
- [runtime: tight loop hangs process completely after some time #15442](https://github.com/golang/go/issues/15442)
- …

Go 语言在 1.14 版本中实现了非协作的抢占式调度，在实现的过程中我们重构已有的逻辑并为 Goroutine 增加新的状态和字段来支持抢占。Go 团队通过下面的一系列提交实现了这一功能，我们可以按时间顺序分析相关提交理解它的工作原理：

- runtime: add general suspendG/resumeG
  - 挂起 Goroutine 的过程是在垃圾回收的栈扫描时完成的，我们通过 [`runtime.suspendG`](https://draveness.me/golang/tree/runtime.suspendG) 和 [`runtime.resumeG`](https://draveness.me/golang/tree/runtime.resumeG) 两个函数重构栈扫描这一过程；
  - 调用 [`runtime.suspendG`](https://draveness.me/golang/tree/runtime.suspendG) 时会将处于运行状态的 Goroutine 的 `preemptStop` 标记成 `true`；
  - 调用 [`runtime.preemptPark`](https://draveness.me/golang/tree/runtime.preemptPark) 可以挂起当前 Goroutine、将其状态更新成 `_Gpreempted` 并触发调度器的重新调度，该函数能够交出线程控制权；
- runtime: asynchronous preemption function for x86
  - 在 x86 架构上增加异步抢占的函数 [`runtime.asyncPreempt`](https://draveness.me/golang/tree/runtime.asyncPreempt) 和 [`runtime.asyncPreempt2`](https://draveness.me/golang/tree/runtime.asyncPreempt2)；
- runtime: use signals to preempt Gs for suspendG
  - 支持通过向线程发送信号的方式暂停运行的 Goroutine；
  - 在 [`runtime.sighandler`](https://draveness.me/golang/tree/runtime.sighandler) 函数中注册 `SIGURG` 信号的处理函数 [`runtime.doSigPreempt`](https://draveness.me/golang/tree/runtime.doSigPreempt)；
  - 实现 [`runtime.preemptM`](https://draveness.me/golang/tree/runtime.preemptM)，它可以通过 `SIGURG` 信号向线程发送抢占请求；
- runtime: implement async scheduler preemption
  - 修改 [`runtime.preemptone`](https://draveness.me/golang/tree/runtime.preemptone) 函数的实现，加入异步抢占的逻辑；

目前的抢占式调度也只会在垃圾回收扫描任务时触发，我们可以梳理一下上述代码实现的抢占式调度过程：

1. 程序启动时，在 [`runtime.sighandler`](https://draveness.me/golang/tree/runtime.sighandler) 中注册 `SIGURG` 信号的处理函数 [`runtime.doSigPreempt`](https://draveness.me/golang/tree/runtime.doSigPreempt)；
2. 在触发垃圾回收的栈扫描时会调用`runtime.suspendG`挂起 Goroutine，该函数会执行下面的逻辑：
   1. 将 `_Grunning` 状态的 Goroutine 标记成可以被抢占，即将 `preemptStop` 设置成 `true`；
   2. 调用 [`runtime.preemptM`](https://draveness.me/golang/tree/runtime.preemptM) 触发抢占；
3. [`runtime.preemptM`](https://draveness.me/golang/tree/runtime.preemptM) 会调用 [`runtime.signalM`](https://draveness.me/golang/tree/runtime.signalM) 向线程发送信号 `SIGURG`；
4. 操作系统会中断正在运行的线程并执行预先注册的信号处理函数 [`runtime.doSigPreempt`](https://draveness.me/golang/tree/runtime.doSigPreempt)；
5. [`runtime.doSigPreempt`](https://draveness.me/golang/tree/runtime.doSigPreempt) 函数会处理抢占信号，获取当前的 SP 和 PC 寄存器并调用 [`runtime.sigctxt.pushCall`](https://draveness.me/golang/tree/runtime.sigctxt.pushCall)；
6. [`runtime.sigctxt.pushCall`](https://draveness.me/golang/tree/runtime.sigctxt.pushCall) 会修改寄存器并在程序回到用户态时执行 [`runtime.asyncPreempt`](https://draveness.me/golang/tree/runtime.asyncPreempt)；
7. 汇编指令 [`runtime.asyncPreempt`](https://draveness.me/golang/tree/runtime.asyncPreempt) 会调用运行时函数 [`runtime.asyncPreempt2`](https://draveness.me/golang/tree/runtime.asyncPreempt2)；
8. [`runtime.asyncPreempt2`](https://draveness.me/golang/tree/runtime.asyncPreempt2) 会调用 [`runtime.preemptPark`](https://draveness.me/golang/tree/runtime.preemptPark)；
9. [`runtime.preemptPark`](https://draveness.me/golang/tree/runtime.preemptPark) 会修改当前 Goroutine 的状态到 `_Gpreempted` 并调用 [`runtime.schedule`](https://draveness.me/golang/tree/runtime.schedule) 让当前函数陷入休眠并让出线程，调度器会选择其它的 Goroutine 继续执行；

上述 9 个步骤展示了基于信号的抢占式调度的执行过程。除了分析抢占的过程之外，我们还需要讨论一下抢占信号的选择，提案根据以下的四个原因选择 `SIGURG` 作为触发异步抢占的信号[7](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/#fn:7)；

1. 该信号需要被调试器透传；
2. 该信号不会被内部的 libc 库使用并拦截；
3. 该信号可以随意出现并且不触发任何后果；
4. 我们需要处理多个平台上的不同信号；

STW 和栈扫描是一个可以抢占的安全点（Safe-points），所以 Go 语言会在这里先加入抢占功能[8](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/#fn:8)。基于信号的抢占式调度只解决了垃圾回收和栈扫描时存在的问题，它到目前为止没有解决所有问题，但是这种真抢占式调度是调度器走向完备的开始，相信在未来我们会在更多的地方触发抢占。

**参考资料**

- https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/