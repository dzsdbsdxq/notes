XA 是 X/Open CAE Specification (Distributed Transaction Processing)模型，它定义的 TM（Transaction Manager）与 RM（Resource Manager）之间进行通信的接口。

Java中 的 `javax.transaction.xa.XAResource` 定义了 XA 接口，它依赖数据库厂商对 jdbc-driver 的具体实现。

- `mysql-connector-java-5.1.30` 的实现可参 `com.mysql.jdbc.jdbc2.optional.MysqlXAConnection` 类。

在 XA 规范中，数据库充当 RM 角色，应用需要充当 TM 的角色，即生成全局的 txId ，调用 XAResource 接口，把多个本地事务协调为全局统一的分布式事务。

目前 XA 有两种实现：

- 基于一阶段提交( 1PC ) 的**弱** XA 。
- 基于二阶段提交( 2PC ) 的**强** XA 。

**弱 XA**

![弱 XA 的顺序图](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/fca74dec683cdea6b986652feec19958)弱 XA 的顺序图

- 弱 XA 通过去掉 XA 的 Prepare 阶段，以达到减少资源锁定范围而提升并发性能的效果。典型的实现为在一个业务线程中，遍历所有的数据库连接，依次做 commit 或者 rollback 。
- 弱 XA 同本地事务相比，性能损耗低，但在事务提交的执行过程中，若出现网络故障、数据库宕机等预期之外的异常，将会造成数据不一致，且无法进行回滚。

🦅 **解决方案？**

基于弱 XA 的事务无需额外的实现成本，相对容易。目前支持的有：

- MyCAT ，具体的源码解析，可以看看 [《MyCAT 源码分析 —— XA 分布式事务》](http://www.iocoder.cn/MyCAT/xa-distributed-transaction/) 。
- Sharding-Sphere 默认支持。

**强 XA**

![强 XA 的顺序图](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/c8f27f0e1107c328476600c4ed2608ec)

强 XA 的顺序图

- 二阶段提交是 XA 的标准实现。它将分布式事务的提交拆分为 2 个阶段：prepare 和 commit/rollback 。
  - 第一阶段：事务管理器要求每个涉及到事务的数据库预提交(precommit)此操作，并反映是否可以提交。
  - 第二阶段：事务协调器要求每个数据库提交数据，或者回滚数据。
- 开启 XA 全局事务后，所有子事务会按照本地默认的隔离级别锁定资源，并记录 undo 和 redo 日志。然后由 TM 发起 prepare 投票，询问所有的子事务是否可以进行提交：
  - 当所有子事务反馈的结果为 “yes” 时，TM 再发起 commit 。
  - 若其中任何一个子事务反馈的结果为“no”，TM 则发起 rollback 。
  - 如果在 prepare 阶段的反馈结果为 “yes” ，而 commit 的过程中出现宕机等异常时，则在节点服务重启后，可根据 XA recover 再次进行 commit 补偿，以保证数据的一致性。

🦅 **优点？**

- 尽量保证了数据的强一致，实现成本较低，在各大主流数据库都有自己实现，对于 MySQL 是从 5.5 开始支持。

🦅 **缺点？**

- 单点问题：事务管理器在整个流程中扮演的角色很关键，如果其宕机，比如在第一阶段已经完成，在第二阶段正准备提交的时候事务管理器宕机，资源管理器就会一直阻塞，导致数据库无法使用。

  > 如果事务管理器是 Proxy 模式的数据库中间件，并且实现高可用，可能可以解决这个问题。不太肯定，需要到时翻下 Sharding Sphere 的源码。TODO

- 同步阻塞：在准备就绪之后，资源管理器中的资源一直处于阻塞，直到提交完成，释放资源。

- 数据不一致：两阶段提交协议虽然为分布式数据强一致性所设计，但仍然存在数据不一致性的可能，比如在第二阶段中，假设协调者发出了事务commit 的通知，但是因为网络问题该通知仅被一部分参与者所收到并执行了 commit 操作，其余的参与者则因为没有收到通知一直处于阻塞状态，这时候就产生了数据的不一致性。

  > 此处的数据不一致也问题不大，因为使用 xa 会锁定记录，无法被访问。

🦅 解决方案？

- Sharding Sphere

  > Sharding Sphere 支持基于 XA 的强一致性事务解决方案，可以通过 SPI 注入不同的第三方组件作为事务管理器实现 XA 协议，如 Atomikos 和 Narayana 。

- [Spring JTA + Atomikos](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-jta.html)

**应用场景**

这种分布式事务方案，比较适合单块应用里，跨多个库的分布式事务，而且因为严重依赖于数据库层面来搞定复杂的事务，效率很低，绝对不适合高并发的场景。

这个方案，我们很少用，一般来说**某个系统内部如果出现跨多个库**的这么一个操作，是**不合规**的。我可以给大家介绍一下， 现在微服务，一个大的系统分成几百个服务，几十个服务。一般来说，我们的规定和规范，是要求**每个服务只能操作自己对应的一个数据库**。

如果你要操作别的服务对应的库，不允许直连别的服务的库，违反微服务架构的规范，你随便交叉胡乱访问，几百个服务的话，全体乱套，这样的一套服务是没法管理的，没法治理的，可能会出现数据被别人改错，自己的库被别人写挂等情况。

如果你要操作别人的服务的库，你必须是通过**调用别的服务的接口**来实现，绝对不允许交叉访问别人的数据库。

![distributed-transacion-XA](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/baef17d9dde837632e9a7894cb825cb2)

<center>distributed-transacion-XA</center>