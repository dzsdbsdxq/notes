> **题目序号：**（1488）
>
> **题目来源**：字节 
>
> **频次**：1

**答案1：**（peace）

- 读channel的时候判断其是否已经关闭
  ```_,ok := <- jobs```
  此时如果 channel 关闭，ok 值为 false

- 写入channel的时候判断其是否已经关闭

1.```_,ok := <- jobs```
此时如果 channel 关闭，ok 值为 false，如果 channel 没有关闭，则会漏掉一个 jobs中的一个数据

2.使用 select 方式
再创建一个 channel，叫做 timeout，如果超时往这个 channel 发送 true，在生产者发送数据给 jobs 的 channel，用 select 监听 timeout，如果超时则关闭 jobs 的 channel。