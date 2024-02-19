
**G（Goroutine）：**我们所说的协程，为用户级的轻量级线程，每个 Goroutine 对象中的 sched 保存着其上下文信息。

**M（Machine）：**对内核级线程的封装，数量对应真实的 CPU 数（真正干活的对 象）。 

**P（Processor）**：即为 G 和 M 的调度对象，用来调度 G 和 M 之间的关联关系， 其数量可通过 GOMAXPROCS()来设置，默认为核心数。