线程没有获取到锁时常见有2种处理方式：

-   一种是没有获取到锁的线程就一直循环等待判断该资源是否已经释放锁，这种锁也叫做**自旋锁**，它不用将线程阻塞起来， 适用于并发低且程序执行时间短的场景，缺点是cpu占用较高
-   另外一种处理方式就是把自己阻塞起来，会**释放CPU给其他线程**，内核会将线程置为「睡眠」状态，等到锁被释放后，内核会在合适的时机唤醒该线程，适用于高并发场景，缺点是有线程上下文切换的开销


Go语言中的Mutex实现了自旋与阻塞两种场景，当满足不了自旋条件时，就会进入阻塞

**允许自旋的条件：**

1. 锁已被占用，并且锁不处于饥饿模式。
2. 积累的自旋次数小于最大自旋次数（active_spin=4）。
3. cpu 核数大于 1。
4. 有空闲的 P。 
5. 当前 goroutine 所挂载的 P 下，本地待运行队列为空。

```
if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {  
...
runtime_doSpin()   
continue  
}


func sync_runtime_canSpin(i int) bool {  
if i >= active_spin 
|| ncpu <= 1 
|| gomaxprocs <= int32(sched.npidle+sched.nmspinning)+1 {  
return false  
}  
if p := getg().m.p.ptr(); !runqempty(p) {  
return false  
}  
return true  
}

```

**自旋：**

```
func sync_runtime_doSpin() {
procyield(active_spin_cnt)
}    

```

如果可以进入自旋状态之后就会调用 `runtime_doSpin` 方法进入自旋， `doSpin` 方法会调用 `procyield(30)` 执行30次 `PAUSE` 指令，什么都不做，但是会消耗CPU时间