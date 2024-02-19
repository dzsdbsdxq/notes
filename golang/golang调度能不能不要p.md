> 题目来源：腾讯

答案：树枝

**1.介绍golang调度器中P是什么？**

Processor的简称，处理器，上下文。 

**2.简述p的功能与为什么必须要P**

它的主要用途就是用来执行goroutine的，它维护了一个goroutine队列，即runqueue。Processor是让咱们从N:1调度到M:N调度的重要部分。