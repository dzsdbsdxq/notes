> 题目序号：216
>
> 题目来源：好未来
>
> 频次：1

**答案1：**（小小）

- **数据交流**：当作并发的 buffer 或者 queue，解决生产者 - 消费者问题。多个 goroutine 可以并发当作生产者（Producer）和消费者（Consumer）。
- **数据传递**：一个goroutine将数据交给另一个goroutine，相当于把数据的拥有权托付出去。
- **信号通知**：一个goroutine可以将信号(closing，closed，data ready等)传递给另一个或者另一组goroutine。
- **任务编排**：可以让一组goroutine按照一定的顺序并发或者串行的执行，这就是编排功能。
- **锁机制**：利用channel实现互斥机制。