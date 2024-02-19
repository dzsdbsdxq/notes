关于 ZooKeeper 是什么，不了解的胖友，直接去看 [《精尽 Zookeeper 面试题》](http://svip.iocoder.cn/Zookeeper/Interview/) 。

在基于 Kafka 的分布式消息队列中，ZooKeeper 的作用有：

- 1、Broker 在 ZooKeeper 中的注册。

- 2、Topic 在 ZooKeeper 中的注册。

- 3、Consumer 在 ZooKeeper 中的注册。

- 4、Producer 负载均衡。

  > 主要指的是，Producer 从 Zookeeper 拉取 Topic 元数据，从而能够将消息发送负载均衡到对应 Topic 的分区中。

- 5、Consumer 负载均衡。

- 6、记录消费进度 Offset 。

  > Kafka 已推荐将 consumer 的 Offset 信息保存在 Kafka 内部的 Topic 中。

- 7、记录 Partition 与 Consumer 的关系。

其实，总结起来，就是两类功能：

- Broker、Producer、Consumer 和 Zookeeper 的交互。

  > 对应 1、2、3、5 。

- 相应的状态存储到 Zookeeper 中。

  > 对应 4、6、7 。

详细的每一点，看 [《再谈基于 Kafka 和 ZooKeeper 的分布式消息队列原理》](https://gitbook.cn/books/5bc446269a9adf54c7ccb8bc/index.html) 的 [「Kafka 架构中 ZooKeeper 以怎样的形式存在？」](http://svip.iocoder.cn/Kafka/Interview/#) 小节。