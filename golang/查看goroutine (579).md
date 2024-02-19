> 题目来源: 小米 
>
> 频次: 1

答案：苦痛律动

使用pprof(建议开一个专题讲pprof使用)

```go
package main
 
import (
  "net/http"
  "runtime/pprof"
)
 
var quit chan struct{} = make(chan struct{})
 
func f() {
  <-quit
}
 
func handler(w http.ResponseWriter, r *http.Request) {
  w.Header().Set("Content-Type", "text/plain")
 
  p := pprof.Lookup("goroutine")
  p.WriteTo(w, 1)
}
 
func main() {
  for i := 0; i < 10000; i++ {
    // 开启10000个协程
    go f()
  }
 
  http.HandleFunc("/", handler)
  // 访问http://localhost:11181/，我们就可以得到所有goroutine的信息
  http.ListenAndServe(":11181", nil)
}
```

**参考资料**

https://www.cnblogs.com/wangxusummer/p/4054564.html