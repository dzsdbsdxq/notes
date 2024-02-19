> 题目序号：（1952,918，1901） 
>
> 题目来源：字节跳动，好未来
>
> 频次: 3 

## **答案：**Zbbxd

golang中的sync.Map是并发安全的，其实也就是sync包中golang⾃定义的⼀个名叫Map的结构体。


```go
type Map struct {
// 该锁⽤来保护dirty
mu Mutex
// 存读的数据，因为是atomic.value类型，只读类型，所以它的读是并发安全的
read atomic.Value // readOnly
//包含最新的写⼊的数据，并且在写的时候，会把read 中未被删除的数据拷⻉到该dirty中，因为是普通的map存在并发安全问题，需要⽤到上⾯的mu字段。
dirty map[interface{}]*entry
// 从read读数据的时候，会将该字段+1，当等于len（dirty）的时候，会将dirty拷⻉到read中（从⽽提升读的性能）。
misses int
}
```

其中read的数据结构为：

```go
type readOnly struct {
m map[interface{}]*entry
// 如果Map.dirty的数据和m 中的数据不⼀样是为true
amended bool
}
```

其中entry的数据结构为：

```go
type entry struct {
//可⻅value是个指针类型，虽然read和dirty存在冗余情况（amended=false），但是由于是指针类型，存储的空间应该不是问题
p unsafe.Pointer // *interface{}
}
```

sync.Map是通过冗余的两个数据结构(read、 dirty),实现性能的提升。为了提升性能， load、 delete、 store等操作尽量使⽤只读的read；为了提⾼read的key击中概率，采⽤动态调整，将dirty数据提升为read；对于数据的删除，采⽤延迟标记删除法，只有在提升dirty的时候才删除。