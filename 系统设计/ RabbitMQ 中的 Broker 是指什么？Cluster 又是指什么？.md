Broker ，是指一个或多个 erlang node 的逻辑分组，且 node 上运行着 RabbitMQ 应用程序。
- Cluster ，是在 Broker 的基础之上，增加了 node 之间共享元数据的约束。
vhost 是什么？起什么作用？

vhost 可以理解为虚拟 Broker ，即 mini-RabbitMQ server 。其内部均含有独立的 queue、exchange 和 binding 等，但最最重要的是，其拥有独立的权限系统，可以做到 vhost 范围的用户控制。当然，从 RabbitMQ 的全局角度，vhost 可以作为不同权限隔离的手段（一个典型的例子就是不同的应用可以跑在不同的 vhost 中）。

> 这个，和 Tomcat、Nginx、Apache 的 vhost 是一样的概念。

