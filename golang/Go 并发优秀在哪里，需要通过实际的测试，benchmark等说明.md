> 题目序号：（4517） 
>
> 题目来源：Shopee
>
> 频次: 1 

**答案：Zbbxd**

Go中天然的支持并发，Go允许使用go语句开启一个新的运行期线程，即 goroutine，以一个不同的、新创建的goroutine来执行一个函数。同一个程序中的所有goroutine共享同一个地址空间。
Goroutine非常轻量，除了为之分配的栈空间，其所占用的内存空间微乎其微。并且其栈空间在开始时非常小，之后随着堆存储空间的按需分配或释放而变化。内部实现上，goroutine会在多个操作系统线程上多路复用。如果一个goroutine阻塞了一个操作系统线程，例如：等待输入，这个线程上的其他goroutine就会迁移到其他线程，这样能继续运行。开发者并不需要关心/担心这些细节。
Go语言的并发机制运用起来非常简便，在启动并发的方式上直接添加了语言级的关键字就可以实现，和其他编程语言相比更加轻量。


1. Go语言并发测试
   参考：
   https://blog.csdn.net/SeanBollock1/article/details/79536069?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ETopBlog-1-79536069-blog-102608785.topblog&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ETopBlog-1-79536069-blog-102608785.topblog&utm_relevant_index=1


2. **与java的比较：**
   参考：
   https://blog.csdn.net/weixin_35657099/article/details/114099451?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-8-114099451-blog-79536069.pc_relevant_default&spm=1001.2101.3001.4242.5&utm_relevant_index=11

测试环境：cpu：2.8 GHz 四核Intel Core i7

内存：16 GB 1600 MHz DDR3

jdk版本：1.8

go版本：1.14

测试方法：分别使用golang和java并发执行相同数量的空任务

golang使用goroutine实现，代码如下：

```go
func main() {
    
    count,line := 100*10000,"-------------------------------------"
    runTask(count)
    fmt.Println(line)
    count = 1000*10000
    runTask(count)
    fmt.Println(line)
    count = 10000*10000
    runTask(count)
    
}

func runTask(taskCount int) {

    runtime.GOMAXPROCS(runtime.NumCPU())
    fmt.Println("golang并发测试")
    fmt.Printf("processors=%d
", runtime.NumCPU())
    fmt.Printf("tasks=%d
", taskCount)
    t1 := time.Now()
    for i:=0; i
    go func() {}()

}

//for runtime.NumGoroutine() > 4 {

//fmt.Println("current goroutines:", runtime.NumGoroutine())

//time.Sleep(time.Second)

//}

t2 := time.Now()

fmt.Printf("cost time: %.3fs
", t2.Sub(t1).Seconds())

}

```

java使用线程池实现，代码如下：

```java
public static void main(String[] args) throws Exception {

    int count = 100*10000;

    String line = "-------------------------------------";

    runTask(count);

    System.out.println(line);

    count = 1000*10000;

    runTask(count);

    System.out.println(line);

    count = 10000*10000;

    runTask(count);

}

public static void runTask(int taskCount){

    int d = Runtime.getRuntime().availableProcessors();

    System.out.println("java并发测试");

    System.out.printf("processors=%d 
",d);

    System.out.printf("tasks=%d
",taskCount);

    ExecutorService service = Executors.newFixedThreadPool(d);

    long start = System.currentTimeMillis();

    for (int i=0;i

    service.submit(() -> {});

    }

    service.shutdown();

    long end = System.currentTimeMillis();

    System.out.printf("cost time: %.3fs
", (end-start)/1000f);

}
```

golang测试结果：

```
golang并发测试

processors=8

tasks=1000000

cost time: 0.291s

golang并发测试

processors=8

tasks=10000000

cost time: 3.090s

golang并发测试

processors=8

tasks=100000000

cost time: 34.591s
```

java测试结果：

```
java并发测试

processors=8

tasks=1000000

cost time: 0.313s

java并发测试

processors=8

tasks=10000000

cost time: 6.239s

java并发测试

processors=8

tasks=100000000

Exception in thread "pool-3-thread-1"
```


结论：golang在处理并发上要优于java！

当并发在百万量级时，golang比java快7%，优势不明显；

当并发在千万量级时，golang比java快2倍以上，优势明显；

当并发在1亿时，golang能够在35秒内处理完成，而java则会在数分钟后抛异常导致程序崩溃。