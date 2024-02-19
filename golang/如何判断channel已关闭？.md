> 题目来源：小米
>
> 频次：高频
>
> 整理人：lws

**方式1：通过读chennel实现**

用 select 和 <-ch 来结合判断，ok的结果和含义：
true：读到数据，并且[通道](https://so.csdn.net/so/search?q=%E9%80%9A%E9%81%93&spm=1001.2101.3001.7020)没有关闭。
false：通道关闭，无数据读到。

需要注意：
1.case 的代码必须是 _, ok:= <- ch 的形式，如果仅仅是 <- ch 来判断，是错的逻辑，因为主要通过 ok的值来判断；
2.select 必须要有 default 分支，否则会阻塞函数，我们要保证一定能正常返回；

```go
ch := make(chan int, 10)
	go func() {
		for i:=1;i<10;i++{
			ch <- i
		}
	}()
	time.Sleep(1*time.Second)
	close(ch)
	for i:=0;i<10;i++{
		select {
		case v, ok := <- ch:
			if ok {
				fmt.Println(v)
			}else{
				fmt.Println("关掉了")
			}
		default:
			fmt.Println("没啥事")
	}
}
```

**方式2：通过context**

通过一个 ctx 变量来指明 close 事件，而不是直接去判断 channel 的一个状态.
当ctx.Done()中有值时，则判断channel已经退出。

注意:
select 的 case 一定要先判断 ctx.Done() 事件，否则很有可能先执行了 chan 的操作从而导致 panic 问题；

```go
select {
case <-ctx.Done():
    // ... exit
    return
case v, ok := <-c:
    // do something....
default:
    // do default ....
}
```

#### 