> 题目序号：（1665）
>
> 题目来源：字节跳动
>
> 频次：1

## 答案：FH-Bin

答案：FH-Bin

题解部分：如下图：绑定在P上的local queue是顺序执行的，不存在执行状态的G协程抢占，所以可以无锁访问。

![image-20220413093707609](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/image-20220413093707609.png)

任务窃取也是窃取其他P上等待状态的G协程，所以也可以不用加锁。

![image-20220413093732931](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/image-20220413093732931.png)