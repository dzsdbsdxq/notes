Kafka 的副本机制，是多个 Broker 节点对其他节点的 Topic 分区的日志进行复制。当集群中的某个节点出现故障，访问故障节点的请求会被转移到其他正常节点(这一过程通常叫 Reblance)，Kafka 每个主题的每个分区都有一个主副本以及 0 个或者多个副本，副本保持和主副本的数据同步，当主副本出故障时就会被替代。

![副本机制](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/1f42986437f76c108007e86a51ae1287)

副本机制

> 注意哈，下面说的 Leader 指的是每个 Topic 的某个分区的 Leader ，而不是 Broker 集群中的【集群控制器】。

在 Kafka 中并不是所有的副本都能被拿来替代主副本，所以在 Kafka 的Leader 节点中维护着一个 ISR（In sync Replicas）集合，翻译过来也叫正在同步中集合，在这个集合中的需要满足两个条件:

- 1、节点必须和 Zookeeper 保持连接。
- 2、在同步的过程中这个副本不能落后主副本太多。

另外还有个 AR（Assigned Replicas）用来标识副本的全集，OSR 用来表示由于落后被剔除的副本集合，所以公式如下：

- ISR = Leader + 没有落后太多的副本。
- AR = OSR + ISR 。

这里先要说下两个名词：HW 和 LEO 。

- HW（高水位 HighWatermark），是 Consumer 能够看到的此 Partition 的位置。
- LEO（logEndOffset），是每个 Partition 的 log 最后一条 Message 的位置。
- HW 能保证 Leader 所在的 Broker 失效，该消息仍然可以从新选举的Leader 中获取，不会造成消息丢失。

当 Producer 向 Leader 发送数据时，可以通过`request.required.acks` 参数来设置数据可靠性的级别：

- 1（默认）：这意味着 Producer 在 ISR 中的 Leader 已成功收到的数据并得到确认后发送下一条 message 。如果 Leader 宕机了，则会丢失数据。
- 0：这意味着 Producer 无需等待来自 Broker 的确认而继续发送下一批消息。这种情况下数据传输效率最高，但是数据可靠性确是最低的。
- -1：Producer 需要等待 ISR 中的所有 Follower 都确认接收到数据后才算一次发送完成，可靠性最高。但是这样也不能保证数据不丢失，比如当 ISR 中只有 Leader 时(其他节点都和 Zookeeper 断开连接，或者都没追上)，这样就变成了 `acks=1` 的情况。

关于这块详详细的内容，推荐阅读

- [《Kafka 数据可靠性深度解读》](https://blog.csdn.net/u013256816/article/details/71091774) 的 「3 高可靠性存储分析」小节
- [《Kafka 集群内复制功能深入剖析》](https://www.jianshu.com/p/03d6a335237f)

