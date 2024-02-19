🦅 **1）cluster**

- cluster 是为了解决当 cluster 中的任意 node 失效后，producer 和 consumer 均可以通过其他 node 继续工作，即提高了可用性；另外可以通过增加 node 数量增加 cluster 的消息吞吐量的目的。
- cluster 本身不负责 message 的可靠性问题（该问题由 producer 通过各种机制自行解决）；cluster 无法解决跨数据中心的问题（即脑裂问题）。
- 另外，在cluster 前使用 HAProxy 可以解决 node 的选择问题，即业务无需知道 cluster 中多个 node 的 ip 地址。可以利用 HAProxy 进行失效 node 的探测，可以作负载均衡。下图为 HAProxy + cluster 的模型：[HAProxy + cluster 的模型](https://img-blog.csdnimg.cn/20181219191433895)

🦅 **2）Mirrored queue**

- Mirrored queue 是为了解决使用 cluster 时所创建的 queue 的完整信息仅存在于单一 node 上的问题，从另一个角度增加可用性。
- 若想正确使用该功能，需要保证：1）consumer 需要支持 Consumer Cancellation Notification 机制；2）consumer 必须能够正确处理重复 message 。

🦅 **3）Warrens**

Warrens 是为了解决 cluster 中 message 可能被 blackholed 的问题，即不能接受 producer 不停 republish message 但 RabbitMQ server 无回应的情况。

Warrens 有两种构成方式：

- 一种模型，是两台独立的 RabbitMQ server + HAProxy ，其中两个 server 的状态分别为 active 和 hot-standby 。该模型的特点为：两台 server 之间无任何数据共享和协议交互，两台 server 可以基于不同的 RabbitMQ 版本。如下图所示：[模型一](https://img-blog.csdnimg.cn/20181219191433914)
- 另一种模型，为两台共享存储的 RabbitMQ server + keepalived ，其中两个 server 的状态分别为 active 和 cold-standby 。该模型的特点为：两台 server 基于共享存储可以做到完全恢复，要求必须基于完全相同的 RabbitMQ 版本。如下图所示：[模型二](https://img-blog.csdnimg.cn/20181219191433931)

Warrens 模型存在的问题：

- 对于第一种模型，虽然理论上讲不会丢失消息，但若在该模型上使用持久化机制，就会出现这样一种情况，即若作为 active 的 server 异常后，持久化在该 server 上的消息将暂时无法被 consume ，因为此时该 queue 将无法在作为 hot-standby 的 server 上被重建，所以，只能等到异常的 active server 恢复后，才能从其上的 queue 中获取相应的 message 进行处理。而对于业务来说，需要具有：a.感知 AMQP 连接断开后重建各种 fabric 的能力；b.感知 active server 恢复的能力；c.切换回 active server 的时机控制，以及切回后，针对 message 先后顺序产生的变化进行处理的能力。
- 对于第二种模型，因为是基于共享存储的模式，所以导致 active server 异常的条件，可能同样会导致 cold-standby server 异常；另外，在该模型下，要求 active 和 cold-standby 的 server 必须具有相同的 node 名和 UID ，否则将产生访问权限问题；最后，由于该模型是冷备方案，故无法保证 cold-standby server 能在你要求的时限内成功启动。