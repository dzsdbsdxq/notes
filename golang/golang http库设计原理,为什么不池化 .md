> **题目序号：**(151, 152)
> **题目来源：** 字节跳动
> **频次:** 1

**答案1：**（阿纪、）

1. **http库设计原理**
   用 Go 实现一个 http server 非常容易，Go 语言标准库 net/http 自带了一系列结构和方法来帮助开发者简化 HTTP 服务开发的相关流程。
   因此，我们不需要依赖任何第三方组件就能构建并启动一个高并发的 HTTP 服务器。
   这里隐去了一些细节，以便了解 Serve 方法的主要逻辑。
   首先创建一个上下文对象，然后调用 Listener 的 Accept() 接收监听到的网络连接；
   一旦有新的连接建立，则调用 Server 的 newConn() 创建新的连接对象，并将连接的状态标志为 StateNew，然后开启一个 goroutine 处理连接请求。

   ```go
   func (srv *Server) Serve(l net.Listener) error {
   ......
      baseCtx := context.Background() // base is always background, per Issue 16220 
      ctx := context.WithValue(baseCtx, ServerContextKey, srv)
      for {
         rw, e := l.Accept()// 接收 listener 过来的网络连接请求
         ......
         c := srv.newConn(rw)
         c.setState(c.rwc, StateNew) // 将连接放在 Server.activeConn这个 map 中
         go c.serve(ctx)// 创建协程处理请求
      }
   }
   ```

2. **为什么不池化?**
   在 C++/Java 实现线程池时，通常可能为了解决创建取消线程开销过大的问题，同时也为不同优先级的请求提供不同的调度模式。
   但是 Golang 已经实现了 M:N 的用户态线程 Goroutine,开销很小。
   对一些特殊的、知道量级的项目，采用goroutine池可能有一定意义，但是大部分情况下不需要，或许也需要严格的 bench。
   用池是可以提升运行效率，降低内存使用的。所以内存吃紧的可以用池优化。不过一般的应用达不到这个量级。