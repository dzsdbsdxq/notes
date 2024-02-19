> 题目序号：（2103）
>
> 题目来源：腾讯
>
> 频次：1

## 答案：郭健 +

Go1.7加入到标准库，在于控制goroutine的生命周期。当一个计算任务被goroutine承接了之后，由于某种原因（超时，或者强制退出）我们希望中止这个goroutine的计算任务，那么就用得到这个Context了。
包含 CancelContext,TimeoutContext,DeadLineContext,ValueContext
**场景一：RPC调用**
在主goroutine上有4个RPC，RPC2/3/4是并行请求的，我们这里希望在RPC2请求失败之后，直接返回错误，并且让RPC3/4停止继续计算。这个时候，就使用的到Context。

```go
package main

import (
	"context"
	"sync"
	"github.com/pkg/errors"
)

func Rpc(ctx context.Context, url string) error {
	result := make(chan int)
	err := make(chan error)

	go func() {
		// 进行RPC调用，并且返回是否成功，成功通过result传递成功信息，错误通过error传递错误信息
		isSuccess := true
		if isSuccess {
			result <- 1
		} else {
			err <- errors.New("some error happen")
		}
	}()

	select {
		case <- ctx.Done():
			// 其他RPC调用调用失败
			return ctx.Err()
		case e := <- err:
			// 本RPC调用失败，返回错误信息
			return e
		case <- result:
			// 本RPC调用成功，不返回错误信息
			return nil
	}
}


func main() {
	ctx, cancel := context.WithCancel(context.Background())

	// RPC1调用
	err := Rpc(ctx, "http://rpc_1_url")
	if err != nil {
		return
	}

	wg := sync.WaitGroup{}

	// RPC2调用
	wg.Add(1)
	go func(){
		defer wg.Done()
		err := Rpc(ctx, "http://rpc_2_url")
		if err != nil {
			cancel()
		}
	}()

	// RPC3调用
	wg.Add(1)
	go func(){
		defer wg.Done()
		err := Rpc(ctx, "http://rpc_3_url")
		if err != nil {
			cancel()
		}
	}()

	// RPC4调用
	wg.Add(1)
	go func(){
		defer wg.Done()
		err := Rpc(ctx, "http://rpc_4_url")
		if err != nil {
			cancel()
		}
	}()

	wg.Wait()
}
```

**场景二：PipeLine**
runSimplePipeline的流水线工人有三个，lineListSource负责将参数一个个分割进行传输，lineParser负责将字符串处理成int64,sink根据具体的值判断这个数据是否可用。他们所有的返回值基本上都有两个chan，一个用于传递数据，一个用于传递错误。（<-chan string, <-chan error）输入基本上也都有两个值，一个是Context，用于传声控制的，一个是(in <-chan)输入产品的。

我们可以看到，这三个工人的具体函数里面，都使用switch处理了case <-ctx.Done()。这个就是生产线上的命令控制。

```go
func lineParser(ctx context.Context, base int, in <-chan string) (	
    <-chan int64, <-chan error, error) {
	...
	go func() {
		defer close(out)
		defer close(errc)

		for line := range in {

			n, err := strconv.ParseInt(line, base, 64)
			if err != nil {
				errc <- err
				return
			}

			select {
			case out <- n:
			case <-ctx.Done():
				return
			}
		}
	}()
	return out, errc, nil
}
```

**场景三：超时请求**
我们发送RPC请求的时候，往往希望对这个请求进行一个超时的限制。当一个RPC请求超过10s的请求，自动断开。当然我们使用CancelContext，也能实现这个功能（开启一个新的goroutine，这个goroutine拿着cancel函数，当时间到了，就调用cancel函数）。

鉴于这个需求是非常常见的，context包也实现了这个需求：timerCtx。具体实例化的方法是 WithDeadline 和 WithTimeout。

具体的timerCtx里面的逻辑也就是通过time.AfterFunc来调用ctx.cancel的。

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
    defer cancel()

    select {
    case <-time.After(1 * time.Second):
        fmt.Println("overslept")
    case <-ctx.Done():
        fmt.Println(ctx.Err()) // prints "context deadline exceeded"
    }
}
```

**场景四：HTTP服务器的request互相传递数据**
context还提供了valueCtx的数据结构。

这个valueCtx最经常使用的场景就是在一个http服务器中，在request中传递一个特定值，比如有一个中间件，做cookie验证，然后把验证后的用户名存放在request中。

我们可以看到，官方的request里面是包含了Context的，并且提供了WithContext的方法进行context的替换。

```go
package main

import (
	"net/http"
	"context"
)

type FooKey string

var UserName = FooKey("user-name")
var UserId = FooKey("user-id")

func foo(next http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		ctx := context.WithValue(r.Context(), UserId, "1")
		ctx2 := context.WithValue(ctx, UserName, "yejianfeng")
		next(w, r.WithContext(ctx2))
	}
}

func GetUserName(context context.Context) string {
	if ret, ok := context.Value(UserName).(string); ok {
		return ret
	}
	return ""
}

func GetUserId(context context.Context) string {
	if ret, ok := context.Value(UserId).(string); ok {
		return ret
	}
	return ""
}

func test(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("welcome: "))
	w.Write([]byte(GetUserId(r.Context())))
	w.Write([]byte(" "))
	w.Write([]byte(GetUserName(r.Context())))
}

func main() {
	http.Handle("/", foo(test))
	http.ListenAndServe(":8080", nil)
}
```

