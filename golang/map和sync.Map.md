> 题目来源：
>
> 频次：1

## 答案：engine

原生map并不支持并发，比如多个goroutine一起对一个map进行读写，会panic，报错：

```
fatal error: concurrent map read and map write
```

当然可以自己实现一个map，封装加sync.RWMutex读写锁，对于每个方法加个锁保护一下。

但是看似没问题，但是有一个问题，那就是效率很低，因为实际上假如两个Write分别对不同的key操作，是不会发生冲突的，没有必要整个map都加读写锁。有必要进行分片加锁，每个锁控制一个分片，从而减少锁的粒度。

一个知名的实现是concurrent-map，它其实就是将map按照key的哈希分为32个小的map，首先通过key可以计算出索引，每次读写的时候先获取小map，然后只针对小的map进行加读写锁，不同小map之间可以完全并行。

官方提供了`sync.Map`，但是一般只是用于特定场景，实际使用的并不多。很多时候还是要自己依据具体的场景开发自己的线程安全锁。

`sync.Map`的场景：

1. 只会增长的缓存系统，一个key只写一次而被读很多次
2. 多个goroutine为不相交的键集读、写和重写键值对

sync.Map的底层结构：

分为两个数据结构：只读的read和可写的dirty，其中read不需要加锁，而dirty写必须要加锁。

```go
type Map struct {
	mu Mutex
  // 只读，不需要锁
	read atomic.Value
  // 必须要加锁
	dirty map[interface{}]*entry
  // 在read中miss的次数
	misses int
}
```

#### 