
**先看下线程安全的定义：**

多个线程访问同一个对象时，调用这个对象的行为都可以获得正确的结果，那么这个对象就是线程安全的。

若有多个线程同时执行写操作，一般都需要考虑线程同步，否则的话就可能影响线程安全。

**再看Go语言实现线程安全常用的几种方式：**

1. 互斥锁
2. 读写锁
3. 原子操作
4. sync.once
5. sync.atomic
6. channel

slice底层结构并没有使用加锁等方式，不支持并发读写，所以并不是线程安全的，使用多个 goroutine 对类型为 slice 的变量进行操作，每次输出的值大概率都不会一样，与预期值不一致;  slice在并发执行中不会报错，但是数据会丢失

```go
/**
* 切片非并发安全
* 多次执行，每次得到的结果都不一样
* 可以考虑使用 channel 本身的特性 (阻塞) 来实现安全的并发读写
*/
func TestSliceConcurrencySafe(t *testing.T) {
a := make([]int, 0)
var wg sync.WaitGroup
for i := 0; i < 10000; i++ {
wg.Add(1)
go func(i int) {
a = append(a, i)
wg.Done()
}(i)
}
wg.Wait()
t.Log(len(a)) 
// not equal 10000
}
```