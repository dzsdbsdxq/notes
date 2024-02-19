> 题目序号：(661)
> 题目来源：网易
> 频次：1

**答案1：**（呼哈）
通过设置最大的可同时使用的 CPU 核数，例如同时执行两个goroutine,设置runtime.GOMAXPROCS(2),用来实现goroutine并行。