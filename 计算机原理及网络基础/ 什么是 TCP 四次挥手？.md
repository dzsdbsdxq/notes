四次挥手，简单来说，就是：

- 发送方：我要和你断开连接！
- 接收方：好的，断吧。
- 接收方：我也要和你断开连接！
- 发送方：好的，断吧。

详细来说，步骤如下：

[![TCP 四次挥手的干货](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/2cf7b8d1689ae6e920ef1b2c3eafb8b2)](http://static.iocoder.cn/2cf7b8d1689ae6e920ef1b2c3eafb8b2)TCP 四次挥手的干货

> 如下使用 Client 和 Server 的方式，仅仅是为了方便，也是可以从 Server 向 Client 发起。

- 第一次挥手：Client 发送一个 `FIN=M` ，用来关闭 Client 到 Server 的数据传送。此时，Client 进入 FIN_WAIT_1 状态。
- 第二次挥手，Server 收到 `FIN` 后，发送一个 `ACK` 给 Client ，确认序号为 `M+1`（与 `SYN` 相同，一个 `FIN` 占用一个序号）。此时，Server 进入 CLOSE_WAIT 状态。注意，TCP 链接处于**半关闭**状态，即客户端已经没有要发送的数据了，但服务端若发送数据，则客户端仍要接收。
- 第三次挥手，Server 发送一个 `FIN=N` ，用来关闭 Server 到 Client 的数据传送。此时 Server 进入 LAST_ACK 状态。
- 第四次挥手，Client 收到 `FIN` 后，此时 Client 进入 TIME_WAIT 状态。接着，Client 发送一个 `ACK` 给 Server ，确认序号为 `N+1` 。Server 接收到后，此时 Server 进入 CLOSED 状态，完成四次挥手。