> 题目来源：好未来

答案：T

详细可以参考：https://blog.csdn.net/tiechui1994/article/details/105752317

首先给出部分参考源码：

```go
type Client struct {	
	Transport RoundTripper 
	CheckRedirect func(req *Request, via []*Request) error
	Jar CookieJar
	Timeout time.Duration
}

type RoundTripper interface {
	RoundTrip(*Request) (*Response, error)
}

type CookieJar interface {
	// SetCookies 在给定URL的回复中处理cookie的接收.
  // 它可能会 或 可能不会 选择保存 Cookie, 具体取决于jar的策略和实现.
	SetCookies(u *url.URL, cookies []*Cookie)

  // Cookies 返回 cookie, 以发送对给定URL的请求. 具体实现取决于标准的cookie使用限制,
  // 例如RFC 6265中的限制.
    	
  Cookies(u *url.URL) []*Cookie
}
```

Transport

+ Client 的 Transport 通常具有内部状态(缓存的TCP连接), 因此 Client 可以被重用. Client 可以
  安全地被多个 goroutine 并发使用.

+ Client 是比 RoundTripper (例如Transport) 更高一级, 并且还处理 HTTP详细信息, 例如 Cookie
  和 Redirect.

Jar和CheckRedirect

- 在进行重定向后, Client 将 Forward(转发) 在初始请求上设置的所有标头, 但以下情况除外:

+ 将 sensitive(敏感的) Header (例如 “Authorization”, “WWW-Authenticate” 和 “Cookie”,
  “Cookie2”) 转发到不受信任的目标时. 当 redirect(重定向) 到一个 “与子域不匹配” 或 “与初始域不完全匹配” 的域时,
  将忽略这些 Header. 例如, 从 “foo.com” 重定向到 “foo.com” 或 “sub.foo.com” 将转发 sensitive Header,
  但重定向到 “bar.com” 则不会.

+ 如果 Jar 非空, 则使用 Jar 转发 “Cookie” 头时, 由于每个重定向可能会更改 CookieJar 的状态,
  因此重定向可能会更改初始请求中设置的 “Cookie” 内容. 当转发 “Cookie” 头时, 任何修改的 Cookie 都将被
  省略, 并希望 Jar 将插入那些修改的 Cookie (假设 origin 匹配).

+ 如果 Jar 为 nil, 则将转发原始 Cookie, 而不进行任何更改.

RoundTripper

+ `RoundTrip` 执行一个 `HTTP事务`, 为提供的 `Request` 返回一个 `Response`.