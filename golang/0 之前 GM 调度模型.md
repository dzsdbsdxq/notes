
调度器把 G 都分配到 M 上，不同的 G 在不同的 M 并发运行时，都需要向系统申 请资源，比如堆栈内存等，因为资源是全局的，就会因为资源竞争照成很多性 能损耗。为了解决这一的问题 go 从 1.1 版本引入，在运行时系统的时候加入 p 对象，让 P 去管理这个 G 对象，M 想要运行 G，必须绑定 P，才能运行 P 所管理 的 G。

GM 调度存在的问题：

1．单一全局互斥锁（Sched.Lock）和集中状态存储

2．Goroutine 传递问题（M 经常在 M 之间传递”可运行”的 goroutine）

3．每个 M 做内存缓存，导致内存占用过高，数据局部性较差 

4．频繁 syscall 调用，导致严重的线程阻塞/解锁，加剧额外的性能损耗。