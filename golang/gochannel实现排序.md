> 题目序号：241
>
> 题目来源: 映客 
>
> 频次 1

**答案1：**（苦痛律动）

应该是使用channel实现一个并归排序

```go
func Merge(ch1 <-chan int, ch2 <-chan int) <-chan int {
 
    out := make(chan int)
 
    go func() {
        // 等上游的数据 （这里有阻塞，和常规的阻塞队列并无不同）
        v1, ok1 := <-ch1
        v2, ok2 := <-ch2
        
        // 取数据
        for ok1 || ok2 {
            if !ok2 || (ok1 && v1 <= v2) {
                // 取到最小值, 就推到 out 中
                out <- v1
                v1, ok1 = <-ch1
            } else {
                out <- v2
                v2, ok2 = <-ch2
            }
        }
        // 显式关闭
        close(out)
    }()
 
    // 开完goroutine后, 主线程继续执行, 不会阻塞
    return out
```

**参考资料**

https://blog.csdn.net/weixin_34122548/article/details/88005287