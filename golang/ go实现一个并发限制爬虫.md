> 题目来源：字节

## 答案：flare

思路使用有缓冲channel与sync.WatitGroup实现并发限制

* 利用有缓冲channel的容量控制并发协程数
* sync.WatitGroup 控制所有任务完成退出