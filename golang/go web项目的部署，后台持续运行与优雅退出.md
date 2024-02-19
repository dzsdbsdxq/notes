> 题目来源：腾讯

答案：村雨

**web项目的部署**
部署 Go 应用相对简单，因为所有应用代码都被打包成一个二进制文件了（视图模板、静态资源和配置文件等非 Go 代码除外），并且不需要依赖其他库（PHP 需要安装各种扩展），不需要额外的运行时环境（比如 Java 需要再安装 JVM），也不需要部署额外的 HTTP 服务器（比如 PHP 还需要再启动 PHP-FPM 处理请求）。

首先，我们可以在本地项目根目录下通过如下命令将应用代码打包成二进制可执行文件：`GOOS=linux GOARCH=amd64 go build`

注意这里指定了 GOOS 和 GOARCH 选项进行交叉编译，我们是在 Mac 系统（amd64）中打包，并且目标二进制文件需要在 Linux 服务器（linux）执行。该命令执行成功后会在当前目录下生成和项目名称相同的二进制文件,之后即可将编译好的二进制文件上传至GitHub等代码托管仓库

在我们的服务器中将GitHub代码下载至本地运行，后台启动二进制文件运行
#####后台持续运行
看起来一切都 OK 了，但是目前这种模式下，用户退出后 Go Web 应用进程会关闭，这显然是不行的，而且如果 Go Web 应用进程因为其他异常挂掉，也无法自动重启，每次需要我们登录到服务器进行启动操作，这很不方便，也影响在线应用的稳定性，为此，我们需要借助第三方进程监控工具帮我们实现 Go Web 应用进程以后台守护进程的方式运行。常见的进程监控工具有 Supervisor、Upstart、systemd 等

**后台优雅退出**
我们编写的Web项目部署之后，经常会因为需要进行配置变更或功能迭代而重启服务，单纯的kill -9 pid的方式会强制关闭进程，这样就会导致服务端当前正在处理的请求失败，那有没有更优雅的方式来实现关机或重启呢？

__什么是优雅关机？__
优雅关机就是服务端关机命令发出后不是立即关机，而是等待当前还在处理的请求全部处理完毕后再退出程序，是一种对客户端友好的关机方式。而执行Ctrl+C关闭服务端时，会强制结束进程导致正在访问的请求出现问题。

__如何实现优雅关机？__
Go 1.8版本之后， http.Server 内置的 Shutdown()方法就支持优雅地关机，具体示例如下：

```go
// +build go1.8
 
package main
 
import (
    "context"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
 
    "github.com/gin-gonic/gin"
)
 
func main() {
    router := gin.Default()
    router.GET("/", func(c *gin.Context) {
        time.Sleep(5 * time.Second)
        c.String(http.StatusOK, "Welcome Gin Server")
    })
 
    srv := &http.Server{
        Addr:    ":8080",
        Handler: router,
    }
 
    go func() {
        // 开启一个goroutine启动服务
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatalf("listen: %s
", err)
        }
    }()
 
    // 等待中断信号来优雅地关闭服务器，为关闭服务器操作设置一个5秒的超时
    quit := make(chan os.Signal, 1) // 创建一个接收信号的通道
    // kill 默认会发送 syscall.SIGTERM 信号
    // kill -2 发送 syscall.SIGINT 信号，我们常用的Ctrl+C就是触发系统SIGINT信号
    // kill -9 发送 syscall.SIGKILL 信号，但是不能被捕获，所以不需要添加它
    // signal.Notify把收到的 syscall.SIGINT或syscall.SIGTERM 信号转发给quit
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)  // 此处不会阻塞
    <-quit  // 阻塞在此，当接收到上述两种信号时才会往下执行
    log.Println("Shutdown Server ...")
    // 创建一个5秒超时的context
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    // 5秒内优雅关闭服务（将未处理完的请求处理完再关闭服务），超过5秒就超时退出
    if err := srv.Shutdown(ctx); err != nil {
        log.Fatal("Server Shutdown: ", err)
    }
 
    log.Println("Server exiting")
}
```

__如何验证优雅关机的效果呢？__

上面的代码运行后会在本地的8080端口开启一个web服务，它只注册了一条路由/，后端服务会先sleep 5秒钟然后才返回响应信息。

我们按下Ctrl+C时会发送syscall.SIGINT来通知程序优雅地关机，具体做法如下：

1.打开终端，编译并执行上面的代码

2.打开一个浏览器，访问127.0.0.1:8080/，此时浏览器白屏等待服务端返回响应。

3.在终端迅速执行Ctrl+C命令给程序发送syscall.SIGINT信号4.此时程序并不立即退出而是等我们第2步的响应返回之后再退出，从而实现优雅关机。

__优雅地重启__

优雅关机实现了，那么该如何实现优雅重启呢？

我们可以使用 fvbock/endless 来替换默认的 ListenAndServe启动服务来实现， 示例代码如下：

```go
package main
 
import (
    "log"
    "net/http"
    "time"
 
    "github.com/fvbock/endless"
    "github.com/gin-gonic/gin"
)
 
func main() {
    router := gin.Default()
    router.GET("/", func(c *gin.Context) {
        time.Sleep(5 * time.Second)
        c.String(http.StatusOK, "hello gin!")
    })
    // 默认endless服务器会监听下列信号：
    // syscall.SIGHUP，syscall.SIGUSR1，syscall.SIGUSR2，syscall.SIGINT，syscall.SIGTERM和syscall.SIGTSTP
    // 接收到 SIGHUP 信号将触发`fork/restart` 实现优雅重启（kill -1 pid会发送SIGHUP信号）
    // 接收到 syscall.SIGINT或syscall.SIGTERM 信号将触发优雅关机
    // 接收到 SIGUSR2 信号将触发HammerTime
    // SIGUSR1 和 SIGTSTP 被用来触发一些用户自定义的hook函数
    if err := endless.ListenAndServe(":8080", router); err!=nil{
        log.Fatalf("listen: %s
", err)
    }
 
    log.Println("Server exiting")
}
```

__如何验证优雅重启的效果呢？__

我们通过执行kill -1 pid命令发送syscall.SIGINT来通知程序优雅重启，具体做法如下：

1.打开终端，go build -o graceful_restart编译并执行./graceful_restart,终端输出当前pid(假设为43682)

2.将代码中处理请求函数返回的hello gin!修改为hello q1mi!，再次编译go build -o graceful_restart

3.打开一个浏览器，访问127.0.0.1:8080/，此时浏览器白屏等待服务端返回响应。

4.在终端迅速执行kill -1 43682命令给程序发送syscall.SIGHUP信号

5.等第3步浏览器收到响应信息hello gin!后再次访问127.0.0.1:8080/会收到hello q1mi!的响应。

6.在不影响当前未处理完请求的同时完成了程序代码的替换，实现了优雅重启。

但是需要注意的是，此时程序的PID变化了，因为endless 是通过fork子进程处理新请求，待原进程处理完当前请求后再退出的方式实现优雅重启的。所以当你的项目是使用类似supervisor的软件管理进程时就不适用这种方式了。

__总结__
无论是优雅关机还是优雅重启归根结底都是通过监听特定系统信号，然后执行一定的逻辑处理保障当前系统正在处理的请求被正常处理后再关闭当前进程。使用优雅关机还是使用优雅重启以及怎么实现，这就需要根据项目实际情况来决定了。