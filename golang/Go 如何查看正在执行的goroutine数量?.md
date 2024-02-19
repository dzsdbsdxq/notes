### 程序中引入pprof pakage

在程序中引入pprof package：

```
import _ "net/http/pprof"
```

程序中开启HTTP监听服务：

```go
package main

import (
"net/http"
_ "net/http/pprof"
)

func main() {

for i := 0; i < 100; i++ {
go func() {
select {}
}()
}

go func() {
http.ListenAndServe("localhost:6060", nil)
}()

select {}
}

```

### 分析goroutine文件

在命令行下执行：

```
go tool pprof -http=:1248 http://127.0.0.1:6060/debug/pprof/goroutine
```

会自动打开浏览器页面如下图所示

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/pprof.png)

在图中可以清晰的看到goroutine的数量以及调用关系，可以看到有103个goroutine