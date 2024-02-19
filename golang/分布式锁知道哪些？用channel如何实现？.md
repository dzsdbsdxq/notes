> 题目序号：(659)
> 题目来源：网易
> 频次：1

**答案1：**（呼哈）

分布式锁定义-控制分布式系统有序的去对共享资源进行操作，通过互斥来保持一致性。
通过数据库，redis，zookeeper都可以实现分布式锁。其中，最常见的是用redis的setnx实现。
通过channel实现：

```go
package main
import ("sync") 

// Lock
type Lock struct {
    c chan struct{}
}

// NewLock
func NewLock() Lock {
    var l Lock
    l.c = make(chan struct{}, 1)
    l.c <- struct{}{}
    return 
}

// Lock, return lock result
func (l Lock) Lock() bool {
    lockResult := false
    select {
        case <-l.c:
        lockResult = true
        default:
    }
    return lockResult
}

// Unlock 
func (l Lock) Unlock() {
	l.c <- struct{}{}
}

var counter int

func main() {
    var l = NewLock()
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            if !l.Lock() {
                // log error
                println("lock failed")
                return
            }
            counter++
            println("current counter", counter)
            l.Unlock()
        }()
    }
    wg.Wait()
}
```


总结：通过channel作为媒介，利用struct{}{}作为信号，判断struct{}{}是否存在进行加锁、解锁操作。