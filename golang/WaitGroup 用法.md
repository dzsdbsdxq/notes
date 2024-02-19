
一个 WaitGroup 对象可以等待一组协程结束。使用方法是：

1. main 协程通过调用 `wg.Add(delta int)` 设置 worker 协程的个数，然后创 建 worker 协程； 
2. worker 协程执行结束以后，都要调用 `wg.Done()`；
3. main 协程调用 `wg.Wait()` 且被 block，直到所有 worker 协程全部执行结束 后返回。