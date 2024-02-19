Go 语言的 `sync.Map` 支持并发读写，采取了 “空间换时间” 的机制，冗余了两个数据结构，分别是：read 和 dirty

```go
type Map struct {
mu Mutex
read atomic.Value // readOnly
dirty map[interface{}]*entry
misses int
}
```

**对比原始map：**

和原始map+RWLock的实现并发的方式相比，减少了加锁对性能的影响。它做了一些优化：可以无锁访问read map，而且会优先操作read map，倘若只操作read map就可以满足要求，那就不用去操作write map(dirty)，所以在某些特定场景中它发生锁竞争的频率会远远小于map+RWLock的实现方式

**优点：**

适合读多写少的场景

**缺点：**

写多的场景，会导致 read map 缓存失效，需要加锁，冲突变多，性能急剧下降