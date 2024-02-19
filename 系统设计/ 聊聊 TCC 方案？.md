TCC 模型是把锁的粒度完全交给业务处理，它需要每个子事务业务都实现Try-Confirm / Cancel 接口。

> TCC 模式本质也是 2PC ，只是 TCC 在应用层控制。

- Try:
  - 尝试执行业务
  - 完成所有业务检查（一致性）
  - 预留必须业务资源（准隔离性）
- Confirm:
  - 确认执行业务；
  - 真正执行业务，不作任何业务检查
  - 只使用Try阶段预留的业务资源
  - Confirm 操作满足幂等性

- Cancel:
  - 取消执行业务
  - 释放Try阶段预留的业务资源
  - Cancel操作满足幂等性

这三个阶段，都会按本地事务的方式执行。不同于 XA的prepare ，TCC 无需将 XA 的投票期间的所有资源挂起，因此极大的提高了吞吐量。

🦅 **应用场景**

下面对TCC模式下，A账户往B账户汇款100元为例子，对业务的改造进行详细的分析：

![img](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/4da5bc0df774ef90e97c6358eb7e632f)

- 汇款服务和收款服务分别需要实现，Try-Confirm-Cancel 接口，并在业务初始化阶段将其注入到 TCC 事务管理器中。

汇款服务

- Try：
  - 检查A账户有效性，即查看A账户的状态是否为“转帐中”或者“冻结”
  - 检查A账户余额是否充足
  - 从A账户中扣减 100 元，并将状态置为“转账中”
  - 预留扣减资源，将从 A 往 B 账户转账 100 元这个事件存入消息或者日志中
- Confirm：
  - 不做任何操作
- Cancel：
  - A 账户增加 100 元
  - 从日志或者消息中，释放扣减资源

收款服务

- Try：
  - 检查 B 账户账户是否有效；
- Confirm：
  - 读取日志或者消息，B 账户增加 100 元
  - 从日志或者消息中，释放扣减资源；
- Cancel：
  - 不做任何操作

由此可以看出，TCC 模型对业务的侵入强，改造的难度大。

但是，在需要前置资源锁定的场景，不得不使用 XA 或 TCC 的方式。再例如说，下单场景，在订单创建之前，需要扣除如下几个资源：

- 优惠劵
- 钱包余额
- 积分

那么，不得不进行前置多资源锁定，无非是使用 XA 的强锁，还是 TCC 的弱锁。在 [oceans](https://github.com/YunaiV/oceans/tree/0.0.1) 的 tag `0.0.1` 中，在未使用 TCC 的情况下，模拟 TCC 的效果的苦闷。

当然，如果能不用 TCC 的情况下，尽量不要用 TCC 。因为，编写回滚逻辑的代码，可能会比较恶心。

🦅 **解决方案？**

- TCC-Transaction ，听说喜马拉雅在用。具体的源码解析，可以看看 [《TCC-Transaction 源码分析》](http://www.iocoder.cn/categories/TCC-Transaction/) 。
- Hmily ，具体的源码解析，可以看看 [《Hmily 实现原理与源码解析系列 —— 精品合集》](http://www.iocoder.cn/Hmily/good-collection/) 。
- ByteTCC