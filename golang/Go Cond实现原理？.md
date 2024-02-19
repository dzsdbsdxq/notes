## 概念

`Go`标准库提供了`Cond`原语，可以让 Goroutine 在满足特定条件时被阻塞和唤醒

## 底层数据结构

```go
type Cond struct {
    noCopy noCopy

    // L is held while observing or changing the condition
    L Locker

    notify  notifyList
    checker copyChecker
}

type notifyList struct {
    wait   uint32
    notify uint32
    lock   uintptr // key field of the mutex
    head   unsafe.Pointer
    tail   unsafe.Pointer
}
```

主要有`4`个字段：

- `nocopy` ： golang 源码中检测禁止拷贝的技术。如果程序中有 WaitGroup 的赋值行为，使用 `go vet` 检查程序时，就会发现有报错，但需要注意的是，noCopy 不会影响程序正常的编译和运行
- `checker`：用于禁止运行期间发生拷贝，双重检查(`Double check`)
- `L`：可以传入一个读写锁或互斥锁，当修改条件或者调用`Wait`方法时需要加锁
- `notify`：通知链表，调用`Wait()`方法的`Goroutine`会放到这个链表中，从这里获取需被唤醒的Goroutine列表

## 使用方法

在Cond里主要有3个方法：

- `sync.NewCond(l Locker)`:  新建一个 sync.Cond 变量，注意该函数需要一个 Locker 作为必填参数，这是因为在 `cond.Wait()` 中底层会涉及到 Locker 的锁操作

- `Cond.Wait()`: 阻塞等待被唤醒，调用Wait函数前**需要先加锁**；并且由于Wait函数被唤醒时存在虚假唤醒等情况，导致唤醒后发现，条件依旧不成立，因此需要使用 for 语句来循环地进行等待，直到条件成立为止

- `Cond.Signal()`: 只唤醒一个最先 Wait 的 goroutine，可以不用加锁
- `Cond.Broadcast()`:  唤醒所有Wait的goroutine，可以不用加锁

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
    go broadcast(c)
    time.Sleep(1 * time.Second)
}

func broadcast(c *sync.Cond) {
    // 原子操作
    atomic.StoreInt64(&status, 1) 
    c.Broadcast()
}

func listen(c *sync.Cond) {
    c.L.Lock()
    for atomic.LoadInt64(&status) != 1 {
        c.Wait() 
        // Wait 内部会先调用 c.L.Unlock()，来先释放锁，如果调用方不先加锁的话，会报错
    }
    fmt.Println("listen")
    c.L.Unlock()
}
```

