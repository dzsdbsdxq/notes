
Golang 在语言级别支持协程，称之为 Goroutine。Golang 标准库提供的所有 系统调用操作(包括所有的同步 I/O 操作)，都会出让 CPU 给其他 Goroutine。这让 Goroutine 的切换管理不依赖于系统的线程和进程，也不依 赖于 CPU 的核心数量，而是交给 Golang 的运行时统一调度。