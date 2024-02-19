> 题目来源：
>
> 频次：1

答案：engine

讲一下RPC基础：

> RPC的概念

RPC（Romote Procedure Call，远程过程调用），作为分布式系统中不同节点之间的通信方式，是分布式系统的基石之一，RPC不是具体的方法，而是一种解决不同服务之间调用的设计。

基于RPC开发的框架可以称为RPC框架，典型的有谷歌的gRPC、阿里的Dubbo、Facebook的Thrift等，当然成熟的RPC框架还会有服务注册与发现、服务治理、负载均衡等功能。

> RPC的四个要素

1. Client

服务调用的发起方

2. Client Stub

用于存储要调用的服务器地址、以及将要请求的数据信息打包，通过网络请求发送给Server Stub，然后阻塞，直到接受到返回的数据，然后进行解析。

3. Server

Server，包含要调用的方法

4. Server Stub

用于接受Client Stub发送的请求数据包并进行解析，完成功能调用，最后将结果进行打包并返回给Client Stub。在没有接受到请求数据包时则处于阻塞状态。

封装了Client Stub和Server Stub后，从Client的角度来看，似乎和本地调用一样。从Server的角度看，似乎就是客户直接调用。

> RPC的具体通信步骤

1. Client以类似本地调用的方式调Client Stub
2. Client Stub序列化生成消息，然后调用本地操作系统的通信模块， Stub阻塞
3. 本地操作系统与远程Server进行通信，消息传输到远程操作系统
4. 远程操作系统将消息传递给Server Stub
5. Server Stub进行反序列化，然后调用Server的对应方法
6. Server程序执行方法，将结果传递给Server Stub
7. Server Stub将结果进行序列化，然后传递给Server操作系统
8. Server操作系统将结果传递给Client
9. Client操作系统将其交给Client Stub， Stub从阻塞状态恢复
10. Client Stub对结果进行反序列化，并将值返回给Client程序
11. Client程序获得返回结果

RPC就是把2-10步进行了封装。

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/v2-619633f1a8ff09c38a0611b1a0d62afc_1440w.jpg)