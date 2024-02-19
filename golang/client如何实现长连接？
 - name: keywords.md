> **题目序号：**(3244)
> **题目来源：** 腾讯
> **频次:** 1

## 答案：重拾

在golang中使用长链接发起HTTP请求，主要依赖Transport,在官方的net/http库中已经支持。

Transport类型实现了RoundTripper接口，支持http、https和http/https代理。Transport类型可以缓存连接以在未来重用。

Transport的主要功能：

- 缓存了长连接，用于大量的http请求场景下的连接复用
- 对连接做一些限制，连接超时时间，每个host的最大连接数。

```go
type RoundTripper interface {
    // RoundTrip执行单次HTTP事务，接收并发挥请求req的回复。
    // RoundTrip不应试图解析/修改得到的回复。
    // 尤其要注意，只要RoundTrip获得了一个回复，不管该回复的HTTP状态码如何，
    // 它必须将返回值err设置为nil。
    // 非nil的返回值err应该留给获取回复失败的情况。
    // 类似的，RoundTrip不能试图管理高层次的细节，如重定向、认证、cookie。
    //
    // 除了从请求的主体读取并关闭主体之外，RoundTrip不应修改请求，包括（请求的）错误。
    // RoundTrip函数接收的请求的URL和Header字段可以保证是（被）初始化了的。
    RoundTrip(*Request) (*Response, error)
}
```

```go
type Transport struct {
    // Proxy指定一个对给定请求返回代理的函数。
    // 如果该函数返回了非nil的错误值，请求的执行就会中断并返回该错误。
    // 如果Proxy为nil或返回nil的*URL置，将不使用代理。
    Proxy func(*Request) (*url.URL, error)
    // Dial指定创建TCP连接的拨号函数。如果Dial为nil，会使用net.Dial。
    Dial func(network, addr string) (net.Conn, error)
    // TLSClientConfig指定用于tls.Client的TLS配置信息。
    // 如果该字段为nil，会使用默认的配置信息。
    TLSClientConfig *tls.Config
    // TLSHandshakeTimeout指定等待TLS握手完成的最长时间。零值表示不设置超时。
    TLSHandshakeTimeout time.Duration
    // 如果DisableKeepAlives为真，会禁止不同HTTP请求之间TCP连接的重用。
    DisableKeepAlives bool
    // 如果DisableCompression为真，会禁止Transport在请求中没有Accept-Encoding头时，
    // 主动添加"Accept-Encoding: gzip"头，以获取压缩数据。
    // 如果Transport自己请求gzip并得到了压缩后的回复，它会主动解压缩回复的主体。
    // 但如果用户显式的请求gzip压缩数据，Transport是不会主动解压缩的。
    DisableCompression bool
    // 如果MaxIdleConnsPerHost!=0，会控制每个主机下的最大闲置连接。
    // 如果MaxIdleConnsPerHost==0，会使用DefaultMaxIdleConnsPerHost。
    MaxIdleConnsPerHost int
    // ResponseHeaderTimeout指定在发送完请求（包括其可能的主体）之后，
    // 等待接收服务端的回复的头域的最大时间。零值表示不设置超时。
    // 该时间不包括获取回复主体的时间。
    ResponseHeaderTimeout time.Duration
    // 内含隐藏或非导出字段
}
```

