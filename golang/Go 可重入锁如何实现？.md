**概念：**

可重入锁又称为递归锁，是指在同一个线程在外层方法获取锁的时候，在进入该线程的内层方法时会自动获取锁，不会因为之前已经获取过还没释放再次加锁导致死锁

**为什么Go语言中没有可重入锁？**

Mutex 不是可重入的锁。Mutex 的实现中没有记录哪个 goroutine 拥有这把锁。理论上，任何 goroutine 都可以随意地 Unlock 这把锁，所以没办法计算重入条件，并且Mutex 重复Lock会导致死锁。

**如何实现可重入锁？**

实现一个可重入锁需要这两点：

-   记住持有锁的线程
-   统计重入的次数

```
package main

import (
"bytes"
"fmt"
"runtime"
"strconv"
"sync"
"sync/atomic"
)

type ReentrantLock struct {
sync.Mutex
recursion int32 // 这个goroutine 重入的次数
owner     int64 // 当前持有锁的goroutine id
}

// Get returns the id of the current goroutine.
func GetGoroutineID() int64 {
var buf [64]byte
var s = buf[:runtime.Stack(buf[:], false)]
s = s[len("goroutine "):]
s = s[:bytes.IndexByte(s, )]
gid, _ := strconv.ParseInt(string(s), 10, 64)
return gid
}

func NewReentrantLock() sync.Locker {
res := &ReentrantLock{
Mutex:     sync.Mutex{},
recursion: 0,
owner:     0,
}
return res
}

// ReentrantMutex 包装一个Mutex,实现可重入
type ReentrantMutex struct {
sync.Mutex
owner     int64 // 当前持有锁的goroutine id
recursion int32 // 这个goroutine 重入的次数
}

func (m *ReentrantMutex) Lock() {
gid := GetGoroutineID()
// 如果当前持有锁的goroutine就是这次调用的goroutine,说明是重入
if atomic.LoadInt64(&m.owner) == gid {
m.recursion++
return
}
m.Mutex.Lock()
// 获得锁的goroutine第一次调用，记录下它的goroutine id,调用次数加1
atomic.StoreInt64(&m.owner, gid)
m.recursion = 1
}

func (m *ReentrantMutex) Unlock() {
gid := GetGoroutineID()
// 非持有锁的goroutine尝试释放锁，错误的使用
if atomic.LoadInt64(&m.owner) != gid {
panic(fmt.Sprintf("wrong the owner(%d): %d!", m.owner, gid))
}
// 调用次数减1
m.recursion--
if m.recursion != 0 { // 如果这个goroutine还没有完全释放，则直接返回
return
}
// 此goroutine最后一次调用，需要释放锁
atomic.StoreInt64(&m.owner, -1)
m.Mutex.Unlock()
}

func main() {
var mutex = &ReentrantMutex{}
mutex.Lock()
mutex.Lock()
fmt.Println(111)
mutex.Unlock()
mutex.Unlock()
}

```

###