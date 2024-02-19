
- `mutexLocked` — 表示互斥锁的锁定状态；
- `mutexWoken` — 表示从正常模式被从唤醒；
- `mutexStarving` — 当前的互斥锁进入饥饿状态； 
- `waitersCount` — 当前互斥锁上等待的 Goroutine 个数；