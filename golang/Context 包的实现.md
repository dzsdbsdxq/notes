> 题目来源：腾讯

答案：千羽

**Context 定义**

context 包中实现了多种 Context 对象。Context 是一个接口，用来描述一个程序的上下文。接口中提供了四个抽象的方法，定义如下：

```go
type Context interface {
  Deadline() (deadline time.Time, ok bool)
  Done() <-chan struct{}
  Err() error
  Value(key interface{}) interface{}
}
```

- Deadline() 返回的是上下文的截至时间，如果没有设定，ok 为 false
- Done() 当执行的上下文被取消后，Done返回的chan就会被close。如果这个上下文不会被取消，返回nil
- Err() 有几种情况:
  - 如果Done() 返回 chan 没有关闭，返回nil
  - 如果Done() 返回的chan 关闭了， Err 返回一个非nil的值，解释为什么会Done()
    - 如果Canceled，返回 "Canceled"
    - 如果超过了 Deadline，返回 "DeadlineEsceeded"
- Value(key) 返回上下文中 key 对应的 value 值

**Context 构造**

为了使用 Context，我们需要了解 Context 是怎么构造的。

Context 提供了两个方法做初始化：

```go
func Background() Context{}
func TODO() Context {}
```

上面方法均会返回空的 Context，但是 Background 一般是所有 Context 的基础，所有 Context 的源头都应该是它。TODO 方法一般用于当传入的方法不确定是哪种类型的 Context 时，为了避免 Context 的参数为nil而初始化的 Context。

其他的 Context 都是基于已经构造好的 Context 来实现的。一个 Context 可以派生多个子 context。基于 Context 派生新Context 的方法如下：

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc){}
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {}
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {}
```

上面三种方法比较类似，均会基于 parent Context 生成一个子 ctx，以及一个 Cancel 方法。如果调用了cancel 方法，ctx 以及基于 ctx 构造的子 context 都会被取消。不同点在于 WithCancel 必需要手动调用 cancel 方法，WithDeadline
可以设置一个时间点，WithTimeout 是设置调用的持续时间，到指定时间后，会调用 cancel 做取消操作。

除了上面的构造方式，还有一类是用来创建传递 traceId， token 等重要数据的 Context。

```go
func WithValue(parent Context, key, val interface{}) Context {}
```

withValue 会构造一个新的context，新的context 会包含一对 Key-Value 数据，可以通过Context.Value(Key) 获取存在 ctx 中的 Value 值。

通过上面的理解可以直到，Context 是一个树状结构，一个 Context 可以派生出多个不一样的Context。我们大概可以画一个如下的树状图：

<img src="https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/20220505232456.jpeg" style="zoom:50%;" />

一个background，衍生出一个带有traceId的valueCtx，然后valueCtx衍生出一个带有cancelCtx
的context。最终在一些db查询，http查询，rpc沙逊等异步调用中体现。如果出现超时，直接把这些异步调用取消，减少消耗的资源，我们也可以在调用时，通过Value 方法拿到traceId，并记录下对应请求的数据。

当然，除了上面的几种 Context 外，我们也可以基于上述的 Context 接口实现新的Context.

原文链接：https://segmentfault.com/a/1190000022534841