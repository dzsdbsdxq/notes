RabbitMQ 使用**发送方确认模式**，确保消息正确地发送到 RabbitMQ。

- 发送方确认模式：将信道设置成 confirm 模式（发送方确认模式），则所有在信道上发布的消息都会被指派一个唯一的 ID 。一旦消息被投递到目的队列后，或者消息被写入磁盘后（可持久化的消息），信道会发送一个确认给生产者（包含消息唯一ID）。如果 RabbitMQ 发生内部错误从而导致消息丢失，会发送一条 nack（not acknowledged，未确认）消息。
- 发送方确认模式是异步的，生产者应用程序在等待确认的同时，可以继续发送消息。当确认消息到达生产者应用程序，生产者应用程序的回调方法就会被触发来处理确认消息。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 RabbitMQ 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「14. 生产者的发送确认」](http://svip.iocoder.cn/RabbitMQ/Interview/#) 小节

🦅 **向不存在的 exchange 发 publish 消息会发生什么？向不存在的 queue 执行 consume 动作会发生什么？**

都会收到 `Channel.Close` 信令告之不存在（内含原因 404 NOT_FOUND）。

🦅 **什么情况下会出现 blackholed 问题？**

> blackholed ，对应中文为“黑洞”。

blackholed 问题是指，向 exchange 投递了 message ，而由于各种原因导致该 message 丢失，但发送者却不知道。可导致 blackholed 的情况：

- 1、向未绑定 queue 的 exchange 发送 message 。
- 2、exchange 以 binding_key key_A 绑定了 queue queue_A，但向该 exchange 发送 message 使用的 routing_key 却是 key_B 。

🦅 **如何防止出现 blackholed 问题？**

没有特别好的办法，只能在具体实践中通过各种方式保证相关 fabric 的存在。另外，如果在执行 `Basic.Publish` 时设置 `mandatory=true` ，则在遇到可能出现 blackholed 情况时，服务器会通过返回 `Basic.Return` 告之当前 message 无法被正确投递（内含原因 312 NO_ROUTE）。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 RabbitMQ 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「14.3 ReturnCallback」](http://svip.iocoder.cn/RabbitMQ/Interview/#) 小节

🦅 **routing_key 和 binding_key 的最大长度是多少？**

255 字节。

🦅 **消息怎么路由？**

从概念上来说，消息路由必须有三部分：交换器、路由、绑定。

- 生产者把消息发布到**交换器**上；

- **绑定**决定了消息如何从路由器路由到特定的队列；

  > 如果一个路由绑定了两个队列，那么发送给该路由时，这两个队列都会增加一条消息。

- 消息最终到达**队列**，并被消费者接收。

详细来说，就是：

- 消息发布到交换器时，消息将拥有一个路由键（routing key），在消息创建时设定。
- 通过队列路由键，可以把队列绑定到交换器上。
- 消息到达交换器后，RabbitMQ 会将消息的路由键与队列的路由键进行匹配（针对不同的交换器有不同的路由规则）。如果能够匹配到队列，则消息会投递到相应队列中；如果不能匹配到任何队列，消息将进入 “黑洞”。

常用的交换器主要分为一下三种：

- direct：如果路由键完全匹配，消息就被投递到相应的队列。
- fanout：如果交换器收到消息，将会广播到所有绑定的队列上。
- topic：可以使来自不同源头的消息能够到达同一个队列。 使用 topic 交换器时，可以使用通配符，比如：`“*”` 匹配特定位置的任意文本， `“.”` 把路由键分为了几部分，`“#”` 匹配所有规则等。特别注意：发往 topic 交换器的消息不能随意的设置选择键（routing_key），必须是由 `"".""` 隔开的一系列的标识符组成。

具体的代码实现，可以看看 [《芋道 Spring Boot 消息队列 RabbitMQ 入门》](http://www.iocoder.cn/Spring-Boot/RabbitMQ/?vip)的[「3. 快速入门」](http://svip.iocoder.cn/RabbitMQ/Interview/#) 小节