
- WaitGroup 主要维护了 2 个计数器，一个是请求计数器 v，一个是等待计数 器 w，二者组成一个 64bit 的值，请求计数器占高 32bit，等待计数器占低 32bit。
- 每次 Add 执行，请求计数器 v 加 1，Done 方法执行，等待计数器减 1，v 为 0 时通过信号量唤醒 Wait()。