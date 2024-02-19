> 题目序号：（4849）
>
> 题目来源：欢聚集团
>
> 频次：1

答案1：（Kjj）

**1、waitGroup对象做值传递**

	如：

```go
func main(){
    var swg sync.WaitGroup
    for i:=0;i<3;i++{
        swg.Add(1)
        go func(wg sync.WaitGroup,mark int){
            defer wg.Done()
            fmt.Printf("%d goroutine finish 
",mark)
        }(swg,i)
    }
    swg.Wait()
}
//运行结果：
//    0 goroutine finish 
//    1 goroutine finish 
//    2 goroutine finish 
//    fatal error: all goroutines are asleep - deadlock!
```

​	子协程中传入的waitGroup对象的一份新值拷贝，在main主协程的waitGroup对象并没有被调用Done（）方法，导致标志位无法被释放，最后发生死锁。

​	所以，将waitGroup对象做参数传递时，使用其引用拷贝传入。

**2、仅在子协程goroutine中进行add操作**

​	如：

```go
func main() {
	wg := sync.WaitGroup{}
	for i := 0; i < 5; i++ {
		go func(wg *sync.WaitGroup, i int) {
			wg.Add(1)
			log.Printf("i:%d", i)
			wg.Done()
		}(&wg, i)
	}
	wg.Wait()
	log.Println("exit")
}
//运行结果（可能出现）：
//    2022/05/09 09:38:36 exit
```

​	因为子协程跟main主协程同步进行，可能子协程中还没来得及add(1)，mian主线程就已经执行结束了。

​	所以，尽量不要仅在子协程中进行add操作。