这四者，对比如下表格如下：

| 特性                     | ActiveMQ                              | RabbitMQ                                           | RocketMQ                                                     | Kafka                                                        |
| :----------------------- | :------------------------------------ | :------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 单机吞吐量               | 万级，比 RocketMQ、Kafka 低一个数量级 | 同 ActiveMQ                                        | 10 万级，支撑高吞吐                                          | 10 万级，高吞吐，一般配合大数据类的系统来进行实时数据计算、日志采集等场景 |
| topic 数量对吞吐量的影响 |                                       |                                                    | topic 可以达到几百/几千的级别，吞吐量会有较小幅度的下降，这是 RocketMQ 的一大优势，在同等机器下，可以支撑大量的 topic | topic 从几十到几百个时候，吞吐量会大幅度下降，在同等机器下，Kafka 尽量保证 topic 数量不要过多，如果要支撑大规模的 topic，需要增加更多的机器资源 |
| 时效性                   | ms 级                                 | 微秒级，这是 RabbitMQ 的一大特点，延迟最低         | ms 级                                                        | 延迟在 ms 级以内                                             |
| 可用性                   | 高，基于主从架构实现高可用            | 同 ActiveMQ                                        | 非常高，分布式架构                                           | 非常高，分布式，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用 |
| 消息可靠性               | 有较低的概率丢失数据                  |                                                    | 经过参数优化配置，可以做到 0 丢失                            | 同 RocketMQ                                                  |
| 功能支持                 | MQ 领域的功能极其完备                 | 基于 erlang 开发，并发能力很强，性能极好，延时很低 | MQ 功能较为完善，还是分布式的，扩展性好                      | 功能较为简单，主要支持简单的 MQ 功能，在大数据领域的实时计算以及日志采集被大规模使用 |

🦅 **ActiveMQ**

一般的业务系统要引入 MQ，最早大家都用 ActiveMQ ，但是现在确实大家用的不多了( 特别是互联网公司 )，没经过大规模吞吐量场景的验证( **性能较差** )，社区也不是很活跃( 主要精力在研发 [ActiveMQ Apollo](https://activemq.apache.org/apollo/) )，所以大家还是算了，我个人不推荐用这个了。

🦅 **RabbitMQ**

后来大家开始用 RabbitMQ，但是确实 Erlang 语言阻止了大量的 Java 工程师去深入研究和掌控它，对公司而言，几乎处于不可控的状态，但是确实人家是开源的，比较稳定的支持，社区活跃度也高。另外，因为 Spring Cloud 在消息队列的支持上，对 RabbitMQ 是比较不错的，所以在选型上又更加被推崇。

🦅 **RocketMQ**

不过现在确实越来越多的公司，会去用 RocketMQ，确实很不错（阿里出品）。~~但社区可能有突然黄掉的风险，对自己公司技术实力有绝对自信的，推荐用 RocketMQ，否则回去老老实实用 RabbitMQ 吧，人家有活跃的开源社区，绝对不会黄。~~ 目前已经加入 Apache ，所以社区层面有相应的保证，并且是使用 Java 语言进行实现，对于 Java 工程师更容易去深入研究和掌控它。目前，也是比较推荐去选择的。并且，如果使用阿里云，可以直接使用其云服务。

当然，现在比较被社区诟病的是，官方暂未提供比较好的中文文档，国内外也缺乏比较好的 RocketMQ 书籍，所以是比较大的痛点。

🦅 **总结**

- 所以**中小型公司**，技术实力较为一般，技术挑战不是特别高，用 RabbitMQ 是不错的选择

- 大型公司

  ，基础架构研发实力较强，用 RocketMQ 是很好的选择。

  - 当然，中小型公司使用 RocketMQ 也是没什么问题的选择，特别是以 Java 为主语言的公司。

- 如果是

  大数据领域

  的实时计算、日志采集等场景，用 Kafka 是业内标准的，绝对没问题，社区活跃度很高，绝对不会黄，何况几乎是全世界这个领域的事实性规范。

  - 另外，目前国内也是有非常多的公司，将 Kafka 应用在业务系统中，例如唯品会、陆金所、美团等等。

目前，艿艿的团队使用 RocketMQ 作为消息队列，因为有 RocketMQ 5 年左右使用经验，并且目前线上环境是使用阿里云，适合我们团队。

🦅 **补充**

推荐阅读如下几篇文章：

- [《Kafka、RabbitMQ、RocketMQ等消息中间件的对比》](https://blog.csdn.net/belvine/article/details/80842240)
- [《Kafka、RabbitMQ、RocketMQ消息中间件的对比 —— 消息发送性能》](http://jm.taobao.org/2016/04/01/kafka-vs-rabbitmq-vs-rocketmq-message-send-performance/)
- [《RocketMQ与Kafka对比（18项差异）》](https://www.cnblogs.com/BYRans/p/6100653.html)

当然，很多测评放在现在已经不适用了，特别是 Kafka ，大量评测是基于 0.X 版本，而 Kafka 目前已经演进到 2.X 版本，已经不可同日而语了。

🔥 **使用示例**

- [《芋道 Spring Boot 消息队列 RocketMQ 入门》](http://www.iocoder.cn/Spring-Boot/RocketMQ/?vip) 对应 [lab-31](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-31) 。
- [《芋道 Spring Boot 消息队列 Kafka 入门》](http://www.iocoder.cn/Spring-Boot/Kafka/?vip) 对应 [lab-03-kafka](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-03-kafka)
- [《芋道 Spring Boot 消息队列 RabbitMQ 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip) 对应 [lab-04-rabbitmq](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq)
- [《芋道 Spring Boot 消息队列 ActiveMQ 入门》](http://www.iocoder.cn/Spring-Boot/ActiveMQ/?vip) 对应 [lab-32](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32) 。

