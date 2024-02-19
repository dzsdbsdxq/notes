![对比图](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/sEDXuxMoHB3wiL5.jpg)

- 轻量级，同样起 web 服务，Nginx 比 Apache 占用更少的内存及资源。
- 抗并发，Nginx 处理请求是异步非阻塞的，而 Apache 则是阻塞型的，在高并发下 Nginx 能保持低资源低消耗高性能。
- 最核心的区别在于 Apache 是同步多进程模型，一个连接对应一个进程；Nginx 是异步的，多个连接（万级别）可以对应一个进程。
- Nginx 高度模块化的设计，编写模块相对简单。