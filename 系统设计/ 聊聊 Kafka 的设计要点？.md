‘1）吞吐量

高吞吐是 Kafka 需要实现的核心目标之一，为此 kafka 做了以下一些设计：

- 1、数据磁盘持久化：消息不在内存中 Cache ，直接写入到磁盘，充分利用磁盘的顺序读写性能。

  > 直接使用 Linux 文件系统的 Cache ，来高效缓存数据。

- 2、zero-copy：减少 IO 操作步骤

  > 采用 Linux Zero-Copy 提高发送性能。
  >
  > - 传统的数据发送需要发送 4 次上下文切换。
  > - 采用 sendfile 系统调用之后，数据直接在内核态交换，系统上下文切换减少为 2 次。根据测试结果，可以提高 60% 的数据发送性能。Zero-Copy 详细的技术细节可以参考 [《Efficient data transfer through zero copy》](https://developer.ibm.com/articles/j-zerocopy/) 文章。

- 3、数据批量发送

- 4、数据压缩

- 5、Topic 划分为多个 Partition ，提高并行度。

  > 数据在磁盘上存取代价为 `O(1)`。
  >
  > - Kafka 以 Topic 来进行消息管理，每个 Topic 包含多个 Partition ，每个 Partition 对应一个逻辑 log ，有多个 segment 文件组成。
  > - 每个 segment 中存储多条消息（见下图），消息 id 由其逻辑位置决定，即从消息 id 可直接定位到消息的存储位置，避免 id 到位置的额外映射。
  > - 每个 Partition 在内存中对应一个 index ，记录每个 segment 中的第一条消息偏移。
  >
  > 发布者发到某个 Topic 的消息会被均匀的分布到多个 Partition 上（随机或根据用户指定的回调函数进行分布），Broker 收到发布消息往对应 Partition 的最后一个 segment 上添加该消息。
  > 当某个 segment上 的消息条数达到配置值或消息发布时间超过阈值时，segment上 的消息会被 flush 到磁盘，只有 flush 到磁盘上的消息订阅者才能订阅到，segment 达到一定的大小后将不会再往该 segment 写数据，Broker 会创建新的 segment 文件。

2）负载均衡

- 1、Producer 根据用户指定的算法，将消息发送到指定的 Partition 中。
- 2、Topic 存在多个 Partition ，每个 Partition 有自己的replica ，每个 replica 分布在不同的 Broker 节点上。多个Partition 需要选取出 Leader partition ，Leader Partition 负责读写，并由 Zookeeper 负责 fail over 。
- 3、相同 Topic 的多个 Partition 会分配给不同的 Consumer 进行拉取消息，进行消费。

3）拉取系统

由于 Kafka Broker 会持久化数据，Broker 没有内存压力，因此， Consumer 非常适合采取 pull 的方式消费数据，具有以下几点好处：

- 1、简化 Kafka 设计。
- 2、Consumer 根据消费能力自主控制消息拉取速度。
- 3、Consumer 根据自身情况自主选择消费模式，例如批量，重复消费，从尾端开始消费等。

4）可扩展性

> 通过 Zookeeper 管理 Broker 与 Consumer 的动态加入与离开。

- 当需要增加 Broker 节点时，新增的 Broker 会向 Zookeeper 注册，而 Producer 及 Consumer 会根据注册在 Zookeeper 上的 watcher 感知这些变化，并及时作出调整。
- 当新增和删除 Consumer 节点时，相同 Topic 的多个 Partition 会分配给剩余的 Consumer 们。

另外，推荐阅读 [《为什么 Kafka 这么快？》](https://mp.weixin.qq.com/s/pzVS7r3QaQPFwob-fY8b4A) 文章，写的更加细致。