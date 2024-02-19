> 题目来源: 字节跳动 
>
> 频次: 1

## 答案：苦痛律动

**阻塞**

我们需要先了解阻塞的概念: 
在执行过程中暂停，以等待某个条件的触发 ，我们就称之为阻塞

**channel**

1. channels用来同步并发执行的函数并提供它们某种传值交流的机制。
2. channels的一些特性：通过channel传递的元素类型、容器（或缓冲区）和传递的方向由“<-”操作符指定。
3. c<-123，把值123输入到管道 c，<-c，把管道 c 的值读取到左边，value :=<-c，这样就是读到 value里面。

**make channel**

```go
// 缓冲
c2 := make(chan int, 3)
// 无缓冲
c1 := make(chan int)
```

在Go中我们make一个channel有两种方式，分别是有缓冲的和没缓冲的

1. 缓冲channel 即 buffer channel 创建方式为 make(chan TYPE,SIZE)
   不仅仅是向 c1 通道放 1，而是一直要等有别的携程 <-c1 接手了这个参数，那么c1<-1才会继续下去，要不然就一直阻塞着。
2. 无缓冲channel 即 unbuffer channel 创建方式为 make(chan TYPE)
   c2<-1 则不会阻塞，因为缓冲大小是3，只有前三个值都还没被人拿走，这时候才会阻塞。

**无缓冲例子**

无缓冲例子1

```go
func test1()  {
    /** 编译错误 deadlock，阻死 main 进程 */
    /** 演示 无缓冲在同一个main里面的 死锁例子 */
    done := make(chan bool)
    done<-true      /** 这句是输入值，它会一直阻塞，等待读取 */
    <-done          /** 这句是读取，但是在上面已经阻死了，永远走不到这里 */
    println("完成")
}
```

无缓冲例子2 (有输入 没读取的死锁)

```go
func test2()  {
    /** 编译错误 deadlock，阻死 main 进程 */
    /** 演示仅有 输入 语句，但没 读取语句 的死锁例子 */
    done := make(chan bool)
    done<-true  /** 输入，一直等待读取，哪怕没读取语句 */
    println("完成")
}
```

无缓冲例子3 (有读取 但没输的死锁)

```go
func test3()  {
    /** 编译错误 deadlock，阻死 main 进程 */
    /** 演示仅有 读取 语句，但没 输入语句 的死锁例子 */
    done := make(chan bool)
    <-done    /** 读取输出，前面没有输入语句，done 是 empty 的，所以一直等待输入 */

    println("完成")
}
```

无缓冲例子4 (协程的阻死，不会影响 main)

```go
func test4()  {
    /** 编译通过 */
    /** 演示，协程的阻死，不会影响 main */
    done := make(chan bool)
    go func() {
        <-done /** 一直等待 */
    }()
    println("完成")
    /**
     * 控制台输出：
     *       完成
     */
}
```

无缓冲例子5 (使用 close 后，不会阻塞)

```go
func test9()  {
    /** 编译通过 */
    /** 演示，没缓存的 channel 使用 close 后，不会阻塞 */
    done := make(chan bool)
    close(done)
    //done<-true  /** 关闭了的，不能再往里面输入值 */
    <-done        /** 这句是读取，但是在上面已经关闭 channel 了，不会阻死 */
    println("完成")
}
```

**有缓冲**

有缓冲例子1 (有输入读取)

```go
func test11()  {
    /** 编译通过 */
    /** 有缓冲的 channel 不会阻塞的例子 */
    done := make(chan bool,1)
    done<-true
    <-done
    println("完成")
}
```

有缓冲例子2 (仅有输入)

```go
func test14()  {
    /** 编译通过 */
    /** 有缓冲的 channel 不会阻塞的例子 */
    done := make(chan bool,1)
    done<-true   /** 不会阻塞在这里，等待读取 */

    println("完成")
}
```

有缓冲例子3 (无输入读取阻塞)

```go
func test12()  {
    /** 编译通过 */
    /** 有缓冲的 channel 会阻塞的例子 */
    done := make(chan bool,1)
    // done<-true /** 注释这句 */
    <-done /** 虽然是有缓冲的，但是在没输入的情况下，读取，会阻塞 */
    println("完成")
}
```

有缓冲例子4 (超过缓冲值未被接受)

```go
func test13()  {
    /** 编译不通过 */
    /** 有缓冲的 channel 会阻塞的例子 */
    done := make(chan bool,1)
    done<-true
    done<-false /** 放第二个值的时候，第一个还没被人拿走，这时候才会阻塞，根据缓冲值而定 */
    println("完成")
}
```

**参考资料**

1. https://blog.csdn.net/qq_36431213/article/details/83281250
2. https://max2d.com/archives/947