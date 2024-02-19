> 题目来源：腾讯
>
> 频次：高频

## 答案：Evan.C

- Map不是线程安全的
- 若想实现map线程安全
  - 方法一：使用读写锁，即**map + sync.RWMutex**
  - 方法二：使用Go提供的**sync.Map**