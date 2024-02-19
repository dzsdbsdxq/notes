Go是一门以并发编程见长的语言，它提供了一系列的同步原语方便开发者使用

## 原子操作

Mutex、RWMutex 等并发原语的底层实现是通过 atomic 包中的一些原子操作来实现的，原子操作是最基础的并发原语

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/53d55255fe851754659d90cbee814f13.jpeg)

```go
package main

import (
    "fmt"
    "sync/atomic"
)

var opts int64 = 0

func main() {
    add(&opts, 3)
    load(&opts)
    compareAndSwap(&opts, 3, 4)
    swap(&opts, 5)
    store(&opts, 6)
}

func add(addr *int64, delta int64) {
    atomic.AddInt64(addr, delta) //加操作
    fmt.Println("add opts: ", *addr)
}

func load(addr *int64) {
    fmt.Println("load opts: ", atomic.LoadInt64(&opts))
}

func compareAndSwap(addr *int64, oldValue int64, newValue int64) {
    if atomic.CompareAndSwapInt64(addr, oldValue, newValue) {
        fmt.Println("cas opts: ", *addr)
        return
    }
}

func swap(addr *int64, newValue int64) {
    atomic.SwapInt64(addr, newValue)
    fmt.Println("swap opts: ", *addr)
}

func store(addr *int64, newValue int64) {
    atomic.StoreInt64(addr, newValue)
    fmt.Println("store opts: ", *addr)
}
```

## Channel

`channel` 管道，高级同步原语，goroutine之间通信的桥梁

使用场景：消息队列、数据传递、信号通知、任务编排、锁

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    c := make(chan struct{}, 1)
    for i := 0; i < 10; i++ {
        go func() {
            c <- struct{}{}
            time.Sleep(1 * time.Second)
            fmt.Println("通过ch访问临界区")
            <-c
        }()
    }
    for {
    }
}
```

## 基本并发原语

Go 语言在 `sync`包中提供了用于同步的一些基本原语，这些基本原语提供了较为基础的同步功能，但是它们是一种相对原始的同步机制，在多数情况下，我们都应该使用抽象层级更高的 Channel 实现同步。

常见的并发原语如下：`sync.Mutex`、`sync.RWMutex`、`sync.WaitGroup`、`sync.Cond`、`sync.Once`、`sync.Pool`、`sync.Context`

### **sync.Mutex**

`sync.Mutex` （互斥锁） 可以限制对临界资源的访问，保证只有一个 goroutine 访问共享资源

使用场景：大量读写，比如多个 goroutine 并发更新同一个资源，像计数器

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    // 封装好的计数器
    var counter Counter
    var wg sync.WaitGroup
    var gNum = 1000
    wg.Add(gNum)
    // 启动10个goroutine
    for i := 0; i < gNum; i++ {
        go func() {
            defer wg.Done()
            counter.Incr() // 受到锁保护的方法
        }()
    }
    wg.Wait()
    fmt.Println(counter.Count())
}

// 线程安全的计数器类型
type Counter struct {
    mu    sync.Mutex
    count uint64
}

// 加1的方法，内部使用互斥锁保护
func (c *Counter) Incr() {
    c.mu.Lock()
    c.count++
    c.mu.Unlock()
}

// 得到计数器的值，也需要锁保护
func (c *Counter) Count() uint64 {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}
```

### **sync.RWMutex**

`sync.RWMutex` （读写锁） 可以限制对临界资源的访问，保证只有一个 goroutine 写共享资源，可以有多个goroutine 读共享资源

使用场景：大量并发读，少量并发写，有强烈的性能要求

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    // 封装好的计数器
    var counter Counter
    var gNum = 1000
    // 启动10个goroutine
    for i := 0; i < gNum; i++ {
        go func() {
            counter.Count() // 受到锁保护的方法
        }()
    }
    for { // 一个writer
        counter.Incr() // 计数器写操作
        fmt.Println("incr")
        time.Sleep(time.Second)
    }
}

// 线程安全的计数器类型
type Counter struct {
    mu    sync.RWMutex
    count uint64
}

// 加1的方法，内部使用互斥锁保护
func (c *Counter) Incr() {
    c.mu.Lock()
    c.count++
    c.mu.Unlock()
}

// 得到计数器的值，也需要锁保护
func (c *Counter) Count() uint64 {
    c.mu.RLock()
    defer c.mu.RUnlock()
    return c.count
}
```

### **sync.WaitGroup**

`sync.WaitGroup` 可以等待一组 Goroutine 的返回

使用场景：并发等待，任务编排，一个比较常见的使用场景是批量发出 RPC 或者 HTTP 请求

```go
requests := []*Request{...}
wg := &sync.WaitGroup{}
wg.Add(len(requests))

for _, request := range requests {
    go func(r *Request) {
        defer wg.Done()
        // res, err := service.call(r)
    }(request)
}
wg.Wait()
```

### **sync.Cond**

 `sync.Cond` 可以让一组的 Goroutine 都在满足特定条件时被唤醒

使用场景：利用等待 / 通知机制实现阻塞或者唤醒

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
    "time"
)

var status int64

func main() {
    c := sync.NewCond(&sync.Mutex{})
    for i := 0; i < 10; i++ {
        go listen(c)
    }
    time.Sleep(1 * time.Second)
    go broadcast(c)
    time.Sleep(1 * time.Second)
}

func broadcast(c *sync.Cond) {
    c.L.Lock()
    atomic.StoreInt64(&status, 1)
    c.Signal()
    c.L.Unlock()
}

func listen(c *sync.Cond) {
    c.L.Lock()
    for atomic.LoadInt64(&status) != 1 {
        c.Wait()
    }
    fmt.Println("listen")
    c.L.Unlock()
}
```

### **sync.Once**

 `sync.Once` 可以保证在 Go 程序运行期间的某段代码只会执行一次

使用场景：常常用于单例对象的初始化场景

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    o := &sync.Once{}
    for i := 0; i < 10; i++ {
        o.Do(func() {
            fmt.Println("only once")
        })
    }
}
```

### **sync.Pool**

`sync.Pool`可以将暂时将不用的对象缓存起来，待下次需要的时候直接使用，不用再次经过内存分配，复用对象的内存，减轻 GC 的压力，提升系统的性能（频繁地分配、回收内存会给 GC 带来一定的负担，严重的时候会引起 CPU 的毛刺）

使用场景：对象池化， TCP连接池、数据库连接池、Worker Pool

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    pool := sync.Pool{
        New: func() interface{} {
            return 0
        },
    }

    for i := 0; i < 10; i++ {
        v := pool.Get().(int)
        fmt.Println(v) // 取出来的值是put进去的，对象复用；如果是新建对象，则取出来的值为0
        pool.Put(i)
    }
}
```

### **sync.Map**

`sync.Map` 线程安全的map

使用场景：map 并发读写

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var scene sync.Map
    // 将键值对保存到sync.Map
    scene.Store("1", 1)
    scene.Store("2", 2)
    scene.Store("3", 3)
    // 从sync.Map中根据键取值
    fmt.Println(scene.Load("1"))
    // 根据键删除对应的键值对
    scene.Delete("1")
    // 遍历所有sync.Map中的键值对
    scene.Range(func(k, v interface{}) bool {
        fmt.Println("iterate:", k, v)
        return true
    })
}
```

### **sync.Context**

`sync.Context` 可以进行上下文信息传递、提供超时和取消机制、控制子 goroutine 的执行

使用场景：取消一个goroutine的执行

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    go func() {
        defer func() {
            fmt.Println("goroutine exit")
        }()
        for {
            select {
            case <-ctx.Done():
                fmt.Println("receive cancel signal!")
                return
            default:
                fmt.Println("default")
                time.Sleep(time.Second)
            }
        }
    }()
    time.Sleep(time.Second)
    cancel()
    time.Sleep(2 * time.Second)
}
```

## 扩展并发原语

### **ErrGroup**

`errgroup` 可以在一组 Goroutine 中提供了同步、错误传播以及上下文取消的功能

使用场景：只要一个 goroutine 出错我们就不再等其他 goroutine 了，减少资源浪费，并且返回错误

```
package main

import (
    "fmt"
    "net/http"

    "golang.org/x/sync/errgroup"
)

func main() {
    var g errgroup.Group
    var urls = []string{
        "http://www.baidu.com/",
        "https://www.sina.com.cn/",
    }
    for i := range urls {
        url := urls[i]
        g.Go(func() error {
            resp, err := http.Get(url)
            if err == nil {
                resp.Body.Close()
            }
            return err
        })
    }
    err := g.Wait()
    if err == nil {
        fmt.Println("Successfully fetched all URLs.")
    } else {
        fmt.Println("fetched error:", err.Error())
    }
}
```

### **Semaphore**

`Semaphore`带权重的信号量，控制多个goroutine同时访问资源

使用场景：控制 goroutine 的阻塞和唤醒

```go
package main

import (
    "context"
    "fmt"
    "log"
    "runtime"
    "time"

    "golang.org/x/sync/semaphore"
)

var (
    maxWorkers = runtime.GOMAXPROCS(0)
    sema       = semaphore.NewWeighted(int64(maxWorkers)) //信号量
    task       = make([]int, maxWorkers*4)

// 任务数，是worker的四
)

func main() {
    ctx := context.Background()
    for i := range task {
        // 如果没有worker可用，会阻塞在这里，直到某个worker被释放
        if err := sema.Acquire(ctx, 1); err != nil {
            break
        }
        // 启动worker goroutine
        go func(i int) {
            defer sema.Release(1)
            time.Sleep(100 * time.Millisecond) // 模拟一个耗时操作
            task[i] = i + 1
        }(i)
    }
    // 请求所有的worker,这样能确保前面的worker都执行完
    if err := sema.Acquire(ctx, int64(maxWorkers)); err != nil {
        log.Printf("获取所有的worker失败: %v", err)
    }
    fmt.Println(maxWorkers, task)
}
```

### **SingleFlight**

用于抑制对下游的重复请求

使用场景：访问缓存、数据库等场景，缓存过期时只有一个请求去更新数据库

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
    "time"

    "golang.org/x/sync/singleflight"
)

// 模拟从数据库读取
func getArticle(id int) (article string, err error) {
    // 假设这里会对数据库进行调用, 模拟不同并发下耗时不同
    atomic.AddInt32(&count, 1)
    time.Sleep(time.Duration(count) * time.Millisecond)

    return fmt.Sprintf("article: %d", id), nil
}

// 模拟优先读缓存，缓存不存在读取数据库，并且只有一个请求读取数据库，其它请求等待
func singleflightGetArticle(sg *singleflight.Group, id int) (string, error) {
    v, err, _ := sg.Do(fmt.Sprintf("%d", id), func() (interface{}, error) {
        return getArticle(id)
    })

    return v.(string), err
}

var count int32

func main() {
    time.AfterFunc(1*time.Second, func() {
        atomic.AddInt32(&count, -count)
    })

    var (
        wg  sync.WaitGroup
        now = time.Now()
        n   = 1000
        sg  = &singleflight.Group{}
    )

    for i := 0; i < n; i++ {
        wg.Add(1)
        go func() {
            res, _ := singleflightGetArticle(sg, 1)
            // res, _ := getArticle(1)
            if res != "article: 1" {
                panic("err")
            }
            wg.Done()
        }()
    }

    wg.Wait()
    fmt.Printf("同时发起 %d 次请求，耗时: %s", n, time.Since(now))
}
```

