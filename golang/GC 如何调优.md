通过 go tool pprof 和 go tool trace 等工具  

- 控制内存分配的速度，限制 Goroutine 的数量，从而提高赋值器对 CPU  的利用率。  
- 减少并复用内存，例如使用 sync.Pool 来复用需要频繁创建临时对象，例 如提前分配足够的内存来降低多余的拷贝。  
- 需要时，增大 GOGC 的值，降低 GC 的运行频率。