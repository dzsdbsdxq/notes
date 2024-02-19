> 题目序号：6341
>
> 题目来源：畅天游
>
> 频次：1

**答案：**（yacoding）

context 监听是否有 IO 操作，开始从当前连接中读取网络请求，每当读取到一个请求则会将该cancelCtx传入，用以传递取消信号，可发送取消信号，取消所有进行中的网络请求。

```go
go func(ctx context.Context, info *Info) {
            timeLimit := 120
            timeoutCtx, cancel := context.WithTimeout(ctx, time.Duration(timeLimit)*time.Millisecond)
            defer func() {
                cancel()
                wg.Done()
            }()
            resp := DoHttp(timeoutCtx, info.req)
        }(ctx, info)
```

关键看业务代码: `resp := DoHttp(timeoutCtx, info.req)` 业务代码中包含`http`调用`NewRequestWithContext`

```go
req, err := http.NewRequestWithContext(ctx, "POST", url, strings.NewReader(paramString))
```

上面的代码，设置了过期时间，当`DoHttp(timeoutCtx, info.req)` 处理时间超过超时时间时，会自动截止，并且打印`context deadline exceeded`。

看个代码：

```go
package main

import (
    "context"
    "fmt"
    "testing"
    "time"
)

func TestTimerContext(t *testing.T) {
    now := time.Now()
    later, _ := time.ParseDuration("10s")

    ctx, cancel := context.WithDeadline(context.Background(), now.Add(later))
    defer cancel()
    go Monitor(ctx)

    time.Sleep(20 * time.Second)

}

func Monitor(ctx context.Context) {
    select {
    case <-ctx.Done():
        fmt.Println(ctx.Err())
    case <-time.After(20 * time.Second):
        fmt.Println("stop monitor")
    }
}
```

Context 接口有如下：

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

- Deadline — 返回 context.Context 被取消的时间，也就是完成工作的截止日期；
- Done — 返回一个 Channel，这个 Channel 会在当前工作完成或者上下文被取消之后关闭，多次调用 Done 方法会返回同一个 Channel；
- Err — 返回 context.Context 结束的原因，它只会在 Done 返回的 Channel 被关闭时才会返回非空的值；
  - 如果 context.Context 被取消，会返回 Canceled 错误；
  - 如果 context.Context 超时，会返回 DeadlineExceeded 错误；

- Value — 从 context.Context 中获取键对应的值，对于同一个上下文来说，多次调用 Value 并传入相同的 Key 会返回相同的结果，该方法可以用来传递请求特定的数据；