> 题目来源：微步

答案：行飞子

golang主要有两种锁：**互斥锁**和**读写锁**

Mutex 可以实现互斥锁，使用互斥锁（Mutex，全称 mutual exclusion）是为了来保护一个资源不会因为并发操作而引起冲突导致数据不准确。

RWMutex可以实现读写锁，每种锁分别对应两个方法，为了避免死锁，两个方法应成对出现，必要时请使用 defer。

- 读锁：调用 RLock 方法开启锁，调用 RUnlock 释放锁
- 写锁：调用 Lock 方法开启锁，调用 Unlock 释放锁（和 Mutex类似）