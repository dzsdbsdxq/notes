**死锁：**

- 单个协程永久阻塞
- 两个或两个以上的协程的执行过程中，由于竞争资源或由于彼此通信而造成的一种阻塞的现象。

**channel死锁场景：**

- 非缓存channel只写不读
- 非缓存channel读在写后面
- 缓存channel写入超过缓冲区数量
- 空读
- 多个协程互相等待

1. 非缓存channel只写不读

```
func deadlock1() {
ch := make(chan int) 
ch <- 3 //  这里会发生一直阻塞的情况，执行不到下面一句
}
```

2. 非缓存channel读在写后面

```
func deadlock2() {
ch := make(chan int)
ch <- 3  //  这里会发生一直阻塞的情况，执行不到下面一句
num := <-ch
fmt.Println("num=", num)
}

func deadlock2() {
ch := make(chan int)
ch <- 100 //  这里会发生一直阻塞的情况，执行不到下面一句
go func() {
num := <-ch
fmt.Println("num=", num)
}()
time.Sleep(time.Second)
}
```

3. 缓存channel写入超过缓冲区数量

```
func deadlock3() {
ch := make(chan int, 3)
ch <- 3
ch <- 4
ch <- 5
ch <- 6  //  这里会发生一直阻塞的情况
}
```

4. 空读

```
func deadlock4() {
ch := make(chan int)
// ch := make(chan int, 1)
fmt.Println(<-ch)  //  这里会发生一直阻塞的情况
}
```

5. **多个协程互相等待**

```
func deadlock5() {
ch1 := make(chan int)
ch2 := make(chan int)
// 互相等对方造成死锁
go func() {
for {
select {
case num := <-ch1:
fmt.Println("num=", num)
ch2 <- 100
}
}
}()

for {
select {
case num := <-ch2:
fmt.Println("num=", num)
ch1 <- 300
}
}
}
```