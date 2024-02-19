> 题目来源：百度

答案：小禾先生

**golang框架图示**

![golang开发框架](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/golang开发框架.png)

**Web框架**

1. gin

	gin是一个知名且简约的Golang Web应用框架。该框架拥有各种开发所需的库合功能。许多知名的开发公司都会采用该 Web 框架来处理各种监控、跟踪、以及调试等问题。此外，相对其他平台，该框架还具有如下特点：

- 非常适合构建出高性能的 REST API。
- 它使用 HTTP 路由器来管理 Golang 流量。
- 它使用简单的涉及规则，并提供精确的文档。

2. beego

​	类似于面向 Python 的兼容性 Django Web 框架，Beego 具有 Web 应用程序共有的一系列独特功能和特性。目前，它由八个不同的模块所组成，可按需取舍或组合使用。

​	除了在大多数 Web 框架中常见的 MVC 组件，Beego 还包括一个 ORM(Object-Relationship Map，对象关系映射)，可访问信息与数据、会话管理工具、以及内置的缓存处理程序。同时，它还包含了用于联合操作和 HTTP 元素、组件、以及各种日志系统的代码库。

​	我们可以认为 Beego 是 Django 在其不同命令行工具中的另一种表示方式。例如，开发人员可以使用 bee 命令，从头开始构建 Beego 应用、或使用当前应用程序进行管理。此外，Beego 还能够提供如下功能：

- 类似于 Django 的 CL(命令行) 工具。
- 从头开始、或在现有的应用中构建强大的应用程序。
- 只需一个 bee 命令，即可全面开展任何项目。

3. martini

​	由于 Martini 可以在整个开发过程中轻松地支持与第三方的集成，因此它更像一个精妙的生态系统，而不是一个框架。除了能够以最小的开销去处理大量功能，该 Web 框架还可以灵活地扩展出其他功能。由于具有极高的可扩展性，因此该框架主要专注于编写 Golang 服务，以及构建出优秀地 Web 应用。此外 Martini 还能够提供如下特点：

- 启用诸如路由、异常处理合常用技术服务，以进一步为通配符、变量参数、以及正则表达式结构等提供支持。
- Martini 拥有非常完备地 Golang Web 应用社区。该社区虽然不大，但是非常活跃，并且目前他们拥有着约20多个实用插件。

4. revel

​	Revel 是面向 Web 开发人员地最新 Golang 框架之一。它能够提供如下功能：

- 它带有一系列预配置地创新特性和功能，可被运用在不同的使用场景中。
- 该框架并不一定需要寻找相关配置与设置。
- 与其他 Go 语言框架不同，Revel 完全自给自足，并不依赖任何中间件或第三方插件。
- Revel 是构建多任务式 API 的一站式解决方案。

5. Goji

​	Goji 是一个极其轻量级和快速的 GolangWeb 开发框架，具有直接组合的能力。目前，该 Web 框架成为了绝大多数移动应用开发公司、以及从事不同 Web 项目公司的理想选择。该框架在如下方面进行了探索：

- 与 net/HTTP ServeMux 类似，拥有简约的 HTTP 请求多路复用器 (multiplexer)。
- Goji 不但适合生产环境，而且包括了各种 URL 模式、可重新配置的中间件栈、以及无缝关闭等功能。

6. Mango

虽然未能得到主动维护，但是许多 Golang 用户仍然会使用到模块化的 Mango Web 框架。Mango 框架可帮助您尽可能轻松快速地去构建和创建可重用的 HTTP 功能模块。此外，它还将一系列应用程序和中间件，包含在一个 HTTP 服务器对象中，以保持代码的自我导向性 (self-directed)。因此，您可以从不同的库中，决定当前项目中需要用到的不同功能。Mango 框架还能够提供如下功能：

- 对于所有类别的 Web 开发项目，Mango 都可以让应用开发人员从各种库选项中进行按需选择，进而简化了应用的实现。
- Mango 框架能够方便开发人员快速、直接地使用基于 HTTP 的模块。
- 为了保持代码的独立性和高效性，它能够与各种应用及中间件协同使用。

7. Gocraft

​	作为老牌稳定的框架，Gocraft 提供了可扩展和快速路由的功能。此类路由可以被作为新的功能，添加到 HTTP 或标准库中的网络包里。由于它是一种定制的 Go mux 中间件包，且具有反射和转换 (casting) 能力，因此您可以将其静态地植入自己的应用代码中。

​	此外，您也可以使用当前的内置中间件，自行创建或添加其他功能。由于程序员往往将性能作为优先考虑因素，因此他们会使用 Gocraft 框架，来轻松地创建和编写后端 Web 应用。因此，Gocraft 混合并提供了如下功能：

- 程序开发人员可以通过具有内置中间件的移动应用，去访问并添加更多的功能。
- Gocraft 可以提供更好的 API 峰值性能。
- 由于支持自定义的中间件包，因此它可以处理代码的反射和转换。

8. echo

​	Echo 是一个极简框架，但它提供了很多最重要的组件。路由器可以将 URL 拆解，然后将拆解的各个部份转换为参数，因此你无需自行解析它们。然后，你可以混合使用身份验证、表单解析、压缩和合理性限制。你可以专注于从函数中返回正确的信息。

9. iris

​	Iris 是一款 Go 语言中用来开发 web 应用的框架，该框架支持编写一次并在任何地方以最小的机器功率运行，如 Android、ios、Linux 和 Windows 等。该框架只需要一个可执行的服务就可以在平台上运行了。

​	Iris以简单而强大的api而闻名。 除了Iris为您提供的低级访问权限。 Iris同样擅长MVC。 它是唯一一个拥有MVC架构模式丰富支持的Go Web框架，性能成本接近于零。

​	Iris为您提供构建面向服务的应用程序的结构。 用Iris构建微服务很容易。

​	在 iris 框架的官方网站上，被称为速度最快的 Go 后端开发框架。在 Iris 的网站文档上，列出了该框架具备的一些特点和框架特性：

- 专注于高性能
- 健壮的静态路由支持和通配符子域名支持
- 视图系统支持超过 5 以上模板
- 支持定制事件的高可扩展性 Websocket API
- 带有 GC, 内存 & redis 提供支持的会话
- 方便的中间件和插件，强大的路由和中间件生态系统
- 完整 REST API，简单流畅的API
- 能定制 HTTP 错误
- 视图系统.支持五种模板隐隐 完全兼容 html/template
- 热重启，源码改变后自动加载

10. negroni

​	有些人看完 Martini 后，决定走一条更简单的道路。他们剥离了路由器和其他一些东西，创建了 Negroni，这是一个非常小型的工具，除了处理标准文件、自定义请求、从基本错误中恢复以及保留日志之外，它不会做更多的工作。如果你想要额外的东西，可以自己加入。Negroni 团队也提供了一系列与可以与 Negroni 一起使用的小型项目。

11. gorilla

​	作为另一个 Google 顶级 Golang 框架，Gorilla 是应用开发社区中最完备的 Web 框架。它完美地迎合了 net/HTTP 库地各种可重用元素合组件。目前 Gorilla 能够提供如下特点：

- 模块化合可扩展性。
- 通过包含合启用新的扩展、模块、删除包，解决过时地功能给系统带来地隐患。
- 涵括从原生支持到对 Web Sockets 的支持。

12. buffalo

​	相比其他 Golang Web 开发框架，Buffalo 不但能够协助您快速地启动开发项目，而且可以被用作集成地 Web 开发生态系统。目前 Buffalo 能够提供如下功能：

- 通过同时满足后端合前端应用开发地需求，实现简单、有效且快速地构建出 Web 应用。
- 带有热重载 (hot reloading) 功能地 Buffalo 框架可以通过 dev 命令，自动观察 .html 和 .go 文件，以重建和重新启动二进制文件。

13. goproxy

​	GoProxy是一款轻量级、功能强大、高性能的http代理、https代理、socks5代理、内网穿透代理服务器、ss代理、游戏盾、游戏代理，支持API代理认证。websocke代理、tcp代理、udp代理、socket代理、高防服务器。支持正向代理、反向代理、透明代理、TCP内网穿透、UDP内网穿透、HTTP内网穿透、HTTPS内网穿透、https代理负载均衡、http代理负载均衡、socks5代理负载均衡、socket代理负载均衡、ss代理负载均衡、TCP/UDP端口映射、SSH中转、TLS加密传输、协议转换、防污染DNS代理，限速，限连接数。

14. go-restful

​	go-restful 是一个 Golang 第三方库，是一个轻量的 RESTful API 框架，基于 Golang Build-in 的 http/net 库。适用于构建灵活多变的 Web Application，Kubernetes 的 ApiServer 也使用了 go-restful。

​	go-restful 具有以下特性：

- 支持可配置的请求路由，默认使用 CurlyRouter 快速路由算法，也支持 RouterJSR311。
- 支持在 URL path 上定义正则表达式，例如：`/static/{subpath:*}`。
- 提供 Request API 用于从 JSON、XML 读取路径参数、查询参数、头部参数，并转换为 Struct。
- 提供 Response API 用于将 Struct 写入到 JSON、XML 以及 Header。
- 支持在服务级、或路由级对请求、响应流进行过滤和拦截。
- 支持使用过滤器自动响应 OPTIONS 请求和 CORS（跨域）请求。
- 支持使用 RecoverHandler 自定义处理 HTTP 500 错误。
- 支持使用 ServiceErrorHandler 自定义处理路由错误产生 HTTP 404/405/406/415 等错误。
- 支持对请求、响应的有效负载进行编码（例如：gzip、deflate）。
- 支持使用 CompressorProvider 注册自定义的 gzip、deflate 的读入器和输出器。
- 支持使用 EntityReaderWriter 注册的自定义编码实现。
- 支持 Swagger UI 编写的 API 文档。
- 支持可配置的日志跟踪。

15. graphql

​	GraphQL 作为一种全新的 API 设计思想，把前端所需要的 API 用类似图数据结构的方式结构清晰地展现出来，让前端很方便地获取所需要的数据。GraphQL 可用来取代目前用的最多的 RESTful 规范，相比于 RESTful，GraphQL 有如下优势：

- 数据的关联性和结构化更好：适合关系性强的图数据查询，例如电影能指向多个演员、演员能指向多部电影；接口即文档，节省手动维护接口文档的精力。
- 更健壮的接口：静态类型 Schema 约束，不仅能校验前端发送的参数是否符合格式，还能自动生成 TypeScript 和 C 等语言中的类型定义。
- 易于前端缓存数据：借助结构化，社区中很多 GraphQL 客户端内置了缓存功能。
- 按需选择：前端根据自己的场景选择部分字段返回，节省计算和网络传输。

**微服务框架**

1. go-kit/kit

​	Go-kit 并不是一个微服务框架，而是一套微服务工具集，我们可以用工具Go-kit为 Go 创建微服务，包含包和接口，有点类似于JAVA Spring Boot，但是没那么强大。可以利用Go-kit提供的API和规范可以创建健壮的，可维护性高的微服务体系，它提供了用于实现系统监控和弹性模式组件的库，例如日志记录、跟踪、限流和熔断等，这些库可以协助开发人员提高微服务架构的性能和稳定性。

2. go-micro

​	Go Micro 是一个基于 Go 语言编写的、用于构建微服务的基础框架，提供了分布式开发所需的核心组件，包括 RPC 和事件驱动通信等。

​	它的设计哲学是「可插拔」的插件化架构，其核心专注于提供底层的接口定义和基础工具，这些底层接口可以兼容各种实现。例如 Go Micro 默认通过 consul 进行服务发现，通过 HTTP 协议进行通信，通过 protobuf 和 json 进行编解码，以便你可以基于这些开箱提供的组件快速启动，但是如果需要的话，你也可以通过符合底层接口定义的其他组件替换默认组件，比如通过 etcd 或 zookeeper 进行服务发现，这也是插件化架构的优势所在：不需要修改任何底层代码即可实现上层组件的替换。

3. goa

​	Goa是一个基于中间件的golang web框架，其整体思想来源于[koajs](https://github.com/koajs/koa)，并且结合了golang的特性。Goa致力于成为 web 应用和 API 开发领域中的一个更轻量、更高效的框架。Goa 并没有捆绑任何中间件，而是提供了一套优雅的方法，其具有如下特性：

- 轻量
- 灵活
- 高效
- 方便易用的API
- 内置错误处理
- 多种格式的处理支持(json、xml、string、html、query、form)

4. kite

​	kite 框架是一个**基于thrift**的**RPC框架**，基于微服务的架构设计，继承了微服务架构具备的各项组件和功能。

​	对于复杂的服务间调用，我们抽象出**五元组**的概念：**(From, FromCluster, To, ToCluster, Method)**。每一个五元组唯一定义了一类的 RPC 调用。以五元组为单元，我们构建了一整套微服务架构。

我们使用 Go 语言研发了内部的微服务框架 kite，协议上完全兼容 Thrift。以五元组为基础单元，我们在 kite 框架上集成了服务注册和发现，分布式负载均衡，超时和熔断管理，服务降级，Method 级别的指标监控，分布式调用链追踪等功能。目前统一使用 kite 框架开发内部 Go 语言的服务，整体架构支持无限制水平扩展。

Kite 的传输层使用 [SockJS](https://github.com/sockjs/sockjs-client) 提供的WebSocket服务， 浏览器Javascript也可以连接到Kite上 （[Kite.js](https://github.com/koding/kite.js)）；

Kite 的RPC消息格式使用修改过的 [dnode ](https://github.com/substack/dnode-protocol)协议，Kite 增加了 session 和 authentication 层, 用于Kites 的发现和识别。

**RPC框架**

grpc-go、grpc-gateway、rpcx

**路由**

httprouter

**数据库**

mysql、gorm、sqlx、gorp、xorm、redigo、redis、mgo、elastic

**命令行**

cobra、urfave/cli、promptui、fatih/color

**日志**

logrus、zap

**网页解析**

goquery

**高级数据结构**

go-datestructures、gods

**GUI**

qt、walk

**App**

go-astilectron

**单元测试**

testify

**错误处理**

pkg/errors

**任务管理**

robfig/cron

**嵌入式设备**

gobot

**文本搜索**

bleve

**分布式计算**

netstack、protoactor-go

**Git**

go-git

**序列化**

golang/protobuf、gogo/protobuf、gopacket、gjson、ffjson、go-simplejson、structs、go-spew

**文件/文本解析**

blackfriday、xlsx、gofpdf、go-yaml/yaml、BurntSushi/toml

**参考**

- https://developer.51cto.com/article/697897.html
- https://juejin.cn/post/6844903743486427150
- https://www.kandaoni.com/article/24
- https://juejin.cn/post/6844903743486427150
- https://www.yelcat.cc/index.php/archives/782/
- https://www.ancii.com/ak6a58xqn/