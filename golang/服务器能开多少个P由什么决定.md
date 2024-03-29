> **题目序号：**487
>
> **题目来源：**跟谁学

**答案1：**（Evan.C）

- P的个数在程序启动时决定，默认情况下等同于CPU的核数
- 程序中可以使用 runtime.GOMAXPROCS() 设置P的个数，在某些IO密集型的场景下可以在一定程度上提高性能。
- 一般来讲，程序运行时就将GOMAXPROCS大小设置为CPU核数，可让Go程序充分利用CPU。在某些IO密集型的应用里，这个值可能并不意味着性能最好。理论上当某个Goroutine进入系统调用时，会有一个新的M被启用或创建，继续占满CPU。但由于Go调度器检测到M被阻塞是有一定延迟的，也即旧的M被阻塞和新的M得到运行之间是有一定间隔的，所以在IO密集型应用中不妨把GOMAXPROCS设置的大一些，或许会有好的效果。