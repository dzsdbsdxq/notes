> 题目序号：87
>
> 题目来源：好未来
>
> 频次：1

答案：咸鱼没有早餐

中间件的设计使得具有一般性、通用性的代码从业务代码中剥离，独立出来。

以网络中的请求响应为例

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/image-20220410184452491.png)

以 go 的原生为例，要实现一个中间件，就要实现 `http.Handler` 接口

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

中间件的一般写成如下形式

```go
func Middleware(h http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // 中间件处理逻辑
        next.ServeHTTP(w, r)
  })
}
```

根据 [Red Hat][https://www.redhat.com/zh/topics/middleware/what-is-middleware] 上将中间件分为五层

**容器层**

中间件的这一层将以统一的方式管理应用生命周期的交付。它提供带有 CI/CD 的 DevOps 能力、容器管理功能以及服务网格功能。

**运行时层**

该层包含了自定义代码的执行环境。中间件可以为高度分布式云环境（例如微服务）、内存中缓存（用于快速访问数据）和消息传递（用于快速数据传输）提供轻量级运行时和框架。

**集成层**

集成中间件可提供相关服务，以通过消息传递、集成和 API 来连接自定义与购买的应用及 SaaS 资产，从而形成功能正常的系统。此外，它还可以提供内存数据库和数据缓存服务、数据/事件流以及 API 管理功能。

**流程自动化和决策管理层**

这是开发中间件的最后一层，旨在强化关键智能，实现优化和自动化，以及加强决策管理。

**工具**

除了上述四层中间件之外，还有相应的应用开发工具。它允许团队使用预设的模板和容器来构建应用，并促进了有效的代码共享和联合开发。这些工具可在本地和云端提供连贯一致的应用开发和交付体验。