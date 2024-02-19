> 题目序号：(1619)
> 题目来源：美团
> 频次：1

**答案1：**（cxiang）

**线程**
`线程`有时被称为`轻量级进程（Lightweight Process）`,是程序执行流的最小单元。

**goroutine** 

goroutine是Go语言中的`轻量级线程`实现，也叫go协程；由Go运行时（runtime）管理

**goroutine 为什么轻量**：

- 资源占用小，每个 goroutine 的初始栈大小仅为 2k；
- 由 Go 运行时而不是操作系统调度，goroutine 上下文切换在用户层完成，开销更小；
- 在语言层面而不是通过标准库提供。goroutine 由go关键字创建，一退出就会被回收或销毁，开发体验更佳
- 语言内置 channel 作为 goroutine 间通信原语，为并发设计提供了强大支撑。