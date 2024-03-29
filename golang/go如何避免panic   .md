> 题目序号：227
>
> 题目来源: 映客 
>
> 频次 1

**答案1：**（苦痛律动）

首先明确panic定义
go把真正的异常叫做 panic，是指出现重大错误，比如数组越界之类的编程BUG或者是那些需要人工介入才能修复的问题，比如程序启动时加载资源出错等等。

几个容易出现panic的点:

1. 函数返回值或参数为指针类型，nil, 未初始化结构体，此时调用容易出现panic，可加 != nil 进行判断
2. 数组切片越界
3. 如果我们关闭未初始化的通道，重复关闭通道，向已经关闭的通道中发送数据，这三种情况也会引发 panic，导致程序崩溃
4. 如果我们直接操作未初始化的映射（map），也会引发 panic，导致程序崩溃
5. 另外，操作映射可能会遇到的更为严重的一个问题是，同时对同一个映射并发读写，它会触发 runtime.throw，不像 panic 可以使用 recover 捕获。所以，我们在对同一个映射并发读写时，一定要使用锁。
6. 如果类型断言使用不当，比如我们不接收布尔值的话，类型断言失败也会引发 panic，导致程序崩溃。


如果很多时候不可避免地出现了panic, 记得使用 defer/recover

**参考资料**

https://cloud.tencent.com/developer/article/1921657