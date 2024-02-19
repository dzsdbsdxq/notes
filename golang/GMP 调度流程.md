
![](https://image-1302243118.cos.ap-beijing.myqcloud.com/recommend/video/blog/13.jpeg)

- 每个 P 有个局部队列，局部队列保存待执行的goroutine（流程 2），当 M 绑定的 P 的的局部队列已经满了之后就会把 goroutine 放到全局队列（流 程 2-1）

- 每个 P 和一个 M 绑定，M 是真正的执行 P 中 goroutine 的实体（流程 3）， M 从绑定的 P 中的局部队列获取 G 来执行

- 当 M 绑定的 P 的局部队列为空时，M 会从全局队列获取到本地队列来执行 G （流程 3.1），当从全局队列中没有获取到可执行的 G 时候，M 会从其他 P 的局部队列中偷取 G 来执行（流程 3.2），这种从其他 P 偷的方式称为 work stealing

- 当 G 因系统调用（syscall）阻塞时会阻塞 M，此时 P 会和 M 解绑即 hand  off，并寻找新的 idle 的 M，若没有 idle 的 M 就会新建一个 M（流程 5.1）

- 当 G 因 channel 或者 network I/O 阻塞时，不会阻塞 M，M 会寻找其他 runnable 的 G；当阻塞的 G 恢复后会重新进入 runnable 进入 P 队列等待执 行（流程 5.3）