答案：呼哈

pprof(performance profiles) - 性能选项)是Go的性能分析工具，在程序运行过程中，可以记录程序的运行信息，可以是CPU使用情况、内存使用情况、goroutine运行情况等，当需要性能调优或者定位Bug时候，这些记录的信息是相当重要。
使用pprof有多种方式，Go已经现成封装好了1个：net/http/pprof，使用简单的几行命令，就可以开启pprof，记录运行信息。
代码参考：

```go
package main
import (
    "net/http"
    _ "net/http/pprof"
    "time"
)
func main() {
    go func() {
        http.ListenAndServe(":6060", nil)
    }()
    time.Sleep(100 * time.Second)
}
CPU profile：go tool pprof http://127.0.0.1:6060/debug/pprof/profile
Heap profile:go tool pprof http://127.0.0.1:6060/debug/pprof/heap
Goroutine profile:go tool pprof http://127.0.0.1:6060/debug/pprof/goroutine
```
