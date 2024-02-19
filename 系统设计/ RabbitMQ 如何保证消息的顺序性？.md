和 Kafka 与 RocketMQ 不同，Kafka 不存在类似类似 Topic 的概念，而是真正的一条一条队列，并且每个队列可以被多个 Consumer 拉取消息。这个，是非常大的一个差异。

🚀 **来看看 RabbitMQ 顺序错乱的场景**：

一个 queue，多个 consumer。比如，生产者向 RabbitMQ 里发送了三条数据，顺序依次是 data1/data2/data3，压入的是 RabbitMQ 的一个内存队列。有三个消费者分别从 MQ 中消费这三条数据中的一条，结果消费者 2 先执行完操作，把 data2 存入数据库，然后是 data1/data3。这不明显乱了。😈 也就是说，乱序消费的问题。

[![乱序](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/29ea655479f826f9a6a5d9d005f46c7c)](http://static.iocoder.cn/29ea655479f826f9a6a5d9d005f46c7c)乱序

🚀 **解决方案**：

- 方案一，拆分多个 queue，每个 queue 一个 consumer，就是多一些 queue 而已，确实是麻烦点。

  > 这个方式，有点模仿 Kafka 和 RocketMQ 中 Topic 的概念。例如说，原先一个 queue 叫 `""xxx""` ，那么多个 queue ，我们可以叫 `""xxx-01""`、`""xxx-02""` 等，相同前缀，不同后缀。

- 方案二，或者就一个 queue 但是对应一个 consumer，然后这个 consumer 内部用内存队列做排队，然后分发给底层不同的 worker 来处理。

  > 这种方式，就是讲一个 queue 里的，相同的“key” 交给同一个 worker 来执行。因为 RabbitMQ 是可以单条消息来 ack ，所以还是比较方便的。这一点，也是和 RocketMQ 和 Kafka 不同的地方。

[![解决乱序](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/f1bd6c79deaa6fae89a62d0e53f1ad43)](http://static.iocoder.cn/f1bd6c79deaa6fae89a62d0e53f1ad43)解决乱序

实际上，我们会发现上述的两个方案，前提都是一个 queue 只能启动一个 consumer 对应。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 RabbitMQ 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「11. 顺序消息」](http://svip.iocoder.cn/RabbitMQ/Interview/#) 小节

