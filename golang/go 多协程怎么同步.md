> 题目来源: 北京合链 
>
> 频次 1

答案：苦痛律动

**通过sync同步**

通过 **sync.WaitGroup** 实现，**WaitGroup**对象内部有一个计数器，最初从0开始, **WaitGroup** 有三个方法
**Add()**: 计数器增加N
**Done()**: 完成一个任务，计数器减少1
**Wait()**: 同步阻塞，计数器为0之后才继续向下执行

```go
var wg sync.WaitGroup
// 完成1-10对公共切片进行赋值并将其值进行平方
var wg sync.WaitGroup
data := make([]int, 10)
for i := 0; i < 10; i++ {
    wg.Add(1)
    go func(idx int, w *sync.WaitGroup) {
        data[idx] = idx
        w.Done()
    }(i, &wg)
}
wg.Wait()
fmt.Println(data)
for j := 0; j < 10; j++ {
    wg.Add(1)
    go func(idx int, w *sync.WaitGroup) {
        data[idx] = int(math.Pow(float64(data[idx]), 2))
        w.Done()
    }(j, &wg)
}
wg.Wait()
fmt.Println(data)

```

**通过channel同步**
例子同上sync例子

```go
ch := make(chan bool, 10)
data := make([]int, 10)
for i := 0; i < 10; i++ {
    go func(idx int) {
        defer func() {
            if err := recover(); err != nil {
                ch <- false
            }
        }()
        data[idx] = idx
        ch <- true
    }(i)
}
fmt.Println(data)
for i := 0; i < 10; i++ {
    go func(idx int) {
        defer func() {
            if err := recover(); err != nil {
                ch <- false
            }
        }()
        data[idx] = int(math.Pow(float64(data[idx]), 2))
        ch <- true
    }(i)
}
for j := 0; j < 10; j++ {
    fmt.Print(<-ch, " ")
}
```

**参考资料**

https://itopic.org/goroutine.html