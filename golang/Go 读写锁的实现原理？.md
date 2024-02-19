**概念：**

读写互斥锁RWMutex，是对Mutex的一个扩展，当一个 goroutine 获得了读锁后，其他 goroutine可以获取读锁，但不能获取写锁；当一个 goroutine 获得了写锁后，其他 goroutine既不能获取读锁也不能获取写锁（只能存在一个写者或多个读者，可以同时读）

**使用场景：**

**读**多于**写**的情况（既保证线程安全，又保证性能不太差）

**底层实现结构:**

互斥锁对应的是底层结构是sync.RWMutex结构体，，位于 src/sync/rwmutex.go中

```
type RWMutex struct {
w           Mutex  // 复用互斥锁
writerSem   uint32 // 信号量，用于写等待读
readerSem   uint32 // 信号量，用于读等待写
readerCount int32  // 当前执行读的 goroutine 数量
readerWait  int32  // 被阻塞的准备读的 goroutine 的数量
}
```


**操作:**

读锁的加锁与释放

```
func (rw *RWMutex) RLock() // 加读锁
func (rw *RWMutex) RUnlock() // 释放读锁
```

**加读锁**

```
func (rw *RWMutex) RLock() {
// 为什么readerCount会小于0呢？往下看发现writer的Lock()会对readerCount做减法操作（原子操作）
if atomic.AddInt32(&rw.readerCount, 1) < 0 {
// A writer is pending, wait for it.
runtime_Semacquire(&rw.readerSem)
}
}
```

`atomic.AddInt32(&rw.readerCount, 1)`  调用这个原子方法，对当前在读的数量加1，如果返回负数，那么说明当前有其他写锁，这时候就调用 `runtime_SemacquireMutex`  休眠当前goroutine 等待被唤醒

**释放读锁**

解锁的时候对正在读的操作减1，如果返回值小于 0 那么说明当前有在写的操作，这个时候调用 `rUnlockSlow`  进入慢速通道

```
func (rw *RWMutex) RUnlock() {
if r := atomic.AddInt32(&rw.readerCount, -1); r < 0 {
rw.rUnlockSlow(r)
}
}
```

被阻塞的准备读的 goroutine 的数量减1，readerWait 为 0，就表示当前没有正在准备读的 goroutine 这时候调用 `runtime_Semrelease`  唤醒写操作

```
func (rw *RWMutex) rUnlockSlow(r int32) {
// A writer is pending.
if atomic.AddInt32(&rw.readerWait, -1) == 0 {
// The last reader unblocks the writer.
runtime_Semrelease(&rw.writerSem, false, 1)
}
}

```

写锁的加锁与释放

```
func (rw *RWMutex) Lock() // 加写锁
func (rw *RWMutex) Unlock() // 释放写锁
```

**加写锁**

```
const rwmutexMaxReaders = 1 << 30

func (rw *RWMutex) Lock() {
// First, resolve competition with other writers.
rw.w.Lock()
// Announce to readers there is a pending writer.
r := atomic.AddInt32(&rw.readerCount, -rwmutexMaxReaders) + rwmutexMaxReaders
// Wait for active readers.
if r != 0 && atomic.AddInt32(&rw.readerWait, r) != 0 {
runtime_Semacquire(&rw.writerSem)
}
}
```

首先调用互斥锁的 lock，获取到互斥锁之后，如果计算之后当前仍然有其他 goroutine 持有读锁，那么就调用 `runtime_SemacquireMutex`  休眠当前的 goroutine 等待所有的读操作完成

这里readerCount 原子性加上一个很大的负数，是防止后面的协程能拿到读锁，阻塞读

**释放写锁**

```
func (rw *RWMutex) Unlock() {
// Announce to readers there is no active writer.
r := atomic.AddInt32(&rw.readerCount, rwmutexMaxReaders)
// Unblock blocked readers, if any.
for i := 0; i < int(r); i++ {
runtime_Semrelease(&rw.readerSem, false)
}
// Allow other writers to proceed.
rw.w.Unlock()
}
```

解锁的操作，会先调用 `atomic.AddInt32(&rw.readerCount, rwmutexMaxReaders)`  将恢复之前写入的负数，然后根据当前有多少个读操作在等待，循环唤醒

**注意点：**

-  读锁或写锁在 Lock() 之前使用 Unlock() 会导致 panic 异常
-  使用 Lock() 加锁后，再次 Lock() 会导致死锁（不支持重入），需Unlock()解锁后才能再加锁
-  锁定状态与 goroutine 没有关联，一个 goroutine 可以 RLock（Lock），另一个 goroutine 可以 RUnlock（Unlock）

**互斥锁和读写锁的区别：**

- 读写锁区分读者和写者，而互斥锁不区分
- 互斥锁同一时间只允许一个线程访问该对象，无论读写；读写锁同一时间内只允许一个写者，但是允许多个读者同时读对象。