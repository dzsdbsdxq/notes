> **题目序号：**(3282)
> **题目来源：** 字节跳动
> **频次:** 1

## 答案：重拾

**使用计数器实现请求限流**

限流的要求是在指定的时间间隔内，server 最多只能服务指定数量的请求。实现的原理是我们启动一个计数器，每次服务请求会把计数器加一，同时到达指定的时间间隔后会把计数器清零；这个计数器的实现代码如下所示：

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"sync"
	"time"
)

type RequestLimitService struct {
	Interval time.Duration
	MaxCount int
	Lock     sync.Mutex
	ReqCount int
}

func NewRequestLimitService(interval time.Duration, maxCnt int) *RequestLimitService {
	reqLimit := &RequestLimitService{
		Interval: interval,
		MaxCount: maxCnt,
	}

	go func() {
		ticker := time.NewTicker(interval)
		for {
			<-ticker.C
			reqLimit.Lock.Lock()
			fmt.Println("Reset Count...")
			reqLimit.ReqCount = 0
			reqLimit.Lock.Unlock()
		}
	}()

	return reqLimit
}

func (reqLimit *RequestLimitService) Increase() {
	reqLimit.Lock.Lock()
	defer reqLimit.Lock.Unlock()

	reqLimit.ReqCount += 1
}

func (reqLimit *RequestLimitService) IsAvailable() bool {
	reqLimit.Lock.Lock()
	defer reqLimit.Lock.Unlock()

	return reqLimit.ReqCount < reqLimit.MaxCount
}

var RequestLimit = NewRequestLimitService(10 * time.Second, 5)

func helloHandler(w http.ResponseWriter, r *http.Request) {
	if RequestLimit.IsAvailable() {
		RequestLimit.Increase()
		fmt.Println(RequestLimit.ReqCount)
		io.WriteString(w, "Hello world!
")
	} else {
		fmt.Println("Reach request limiting!")
		io.WriteString(w, "Reach request limit!
")
	}
}

func main() {
	fmt.Println("Server Started!")
	http.HandleFunc("/", helloHandler)
	http.ListenAndServe(":8000", nil)
}
```

