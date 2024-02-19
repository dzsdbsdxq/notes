> 题目序号：（2104）
>
> 题目来源：腾讯
>
> 频次：1

答案：郭健

我们分析的 Go 版本依然是 1.9.2。

**整体概览**
context 包的代码并不长，context.go 文件总共不到 500 行，其中还有很多大段的注释，代码可能也就 200 行左右的样子，是一个非常值得研究的代码库。

先给大家看一张整体的图：
![avatar](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/59145775-173af280-8a1b-11e9-8867-c99ce02edb09.png)

<div class="table-wrapper"><table>
<thead>
<tr>
<th>类型</th>
<th>名称</th>
<th>作用</th>
</tr>
</thead>
<tbody>
<tr>
<td>Context</td>
<td>接口</td>
<td>定义了 Context 接口的四个方法</td>
</tr>
<tr>
<td>emptyCtx</td>
<td>结构体</td>
<td>实现了 Context 接口，它其实是个空的 context</td>
</tr>
<tr>
<td>CancelFunc</td>
<td>函数</td>
<td>取消函数</td>
</tr>
<tr>
<td>canceler</td>
<td>接口</td>
<td>context 取消接口，定义了两个方法</td>
</tr>
<tr>
<td>cancelCtx</td>
<td>结构体</td>
<td>可以被取消</td>
</tr>
<tr>
<td>timerCtx</td>
<td>结构体</td>
<td>超时会被取消</td>
</tr>
<tr>
<td>valueCtx</td>
<td>结构体</td>
<td>可以存储 k-v 对</td>
</tr>
<tr>
<td>Background</td>
<td>函数</td>
<td>返回一个空的 context，常作为根 context</td>
</tr>
<tr>
<td>TODO</td>
<td>函数</td>
<td>返回一个空的 context，常用于重构时期，没有合适的 context 可用</td>
</tr>
<tr>
<td>WithCancel</td>
<td>函数</td>
<td>基于父 context，生成一个可以取消的 context</td>
</tr>
<tr>
<td>newCancelCtx</td>
<td>函数</td>
<td>创建一个可取消的 context</td>
</tr>
<tr>
<td>propagateCancel</td>
<td>函数</td>
<td>向下传递 context 节点间的取消关系</td>
</tr>
<tr>
<td>parentCancelCtx</td>
<td>函数</td>
<td>找到第一个可取消的父节点</td>
</tr>
<tr>
<td>removeChild</td>
<td>函数</td>
<td>去掉父节点的孩子节点</td>
</tr>
<tr>
<td>init</td>
<td>函数</td>
<td>包初始化</td>
</tr>
<tr>
<td>WithDeadline</td>
<td>函数</td>
<td>创建一个有 deadline 的 context</td>
</tr>
<tr>
<td>WithTimeout</td>
<td>函数</td>
<td>创建一个有 timeout 的 context</td>
</tr>
<tr>
<td>WithValue</td>
<td>函数</td>
<td>创建一个存储 k-v 对的 context</td>
</tr>
</tbody>
</table></div>
**整体类图如下**
![avatar](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/59153629-c1a12d00-8a90-11e9-89a4-eaf3e34f190e.png)

**接口**

```go
type Context interface {	
	// 当 context 被取消或者到了 deadline，返回一个被关闭的 channel
	Done() <-chan struct{}

	// 在 channel Done 关闭后，返回 context 取消原因
	Err() error

	// 返回 context 是否会被取消以及自动取消时间（即 deadline）
	Deadline() (deadline time.Time, ok bool)

	// 获取 key 对应的 value
	Value(key interface{}) interface{}
}
```

Context 是一个接口，定义了 4 个方法，它们都是幂等的。也就是说连续多次调用同一个方法，得到的结果都是相同的。

Done() 返回一个 channel，可以表示 context 被取消的信号：当这个 channel 被关闭时，说明 context 被取消了。注意，这是一个只读的channel。 我们又知道，读一个关闭的 channel 会读出相应类型的零值。并且源码里没有地方会向这个 channel 里面塞入值。换句话说，这是一个 receive-only 的 channel。因此在子协程里读这个 channel，除非被关闭，否则读不出来任何东西。也正是利用了这一点，子协程从 channel 里读出了值（零值）后，就可以做一些收尾工作，尽快退出。

Err() 返回一个错误，表示 channel 被关闭的原因。例如是被取消，还是超时。

Deadline() 返回 context 的截止时间，通过此时间，函数就可以决定是否进行接下来的操作，如果时间太短，就可以不往下做了，否则浪费系统资源。当然，也可以用这个 deadline 来设置一个 I/O 操作的超时时间。

Value() 获取之前设置的 key 对应的 value。

**canceler**

```go
type canceler interface {
    cancel(removeFromParent bool, err error)
	Done() <-chan struct{}
}
```

实现了上面定义的两个方法的 Context，就表明该 Context 是可取消的。源码中有两个类型实现了 canceler 接口：*cancelCtx 和 *timerCtx。注意是加了 * 号的，是这两个结构体的指针实现了 canceler 接口。

Context 接口设计成这个样子的原因：

“取消”操作应该是建议性，而非强制性
caller 不应该去关心、干涉 callee 的情况，决定如何以及何时 return 是 callee 的责任。caller 只需发送“取消”信息，callee 根据收到的信息来做进一步的决策，因此接口并没有定义 cancel 方法。

“取消”操作应该可传递
“取消”某个函数时，和它相关联的其他函数也应该“取消”。因此，Done() 方法返回一个只读的 channel，所有相关函数监听此 channel。一旦 channel 关闭，通过 channel 的“广播机制”，所有监听者都能收到。
**emptyCtx**

```go
type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
	return
}

func (*emptyCtx) Done() <-chan struct{} {
	return nil
}

func (*emptyCtx) Err() error {
	return nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
	return nil
}
```

**cancelCtx**

```go
type cancelCtx struct {	
    Context

	// 保护之后的字段
	mu       sync.Mutex
	done     chan struct{}
	children map[canceler]struct{}
	err      error
}
```

```go
 func (c *cancelCtx) cancel(removeFromParent bool, err error) {
     // 必须要传 err
	if err == nil {
		panic("context: internal error: missing cancel error")
	}
	c.mu.Lock()
	if c.err != nil {
		c.mu.Unlock()
		return // 已经被其他协程取消
	}
	// 给 err 字段赋值
	c.err = err
	// 关闭 channel，通知其他协程
	if c.done == nil {
		c.done = closedchan
	} else {
		close(c.done)
	}
	
	// 遍历它的所有子节点
	for child := range c.children {
	    // 递归地取消所有子节点
		child.cancel(false, err)
	}
	// 将子节点置空
	c.children = nil
	c.mu.Unlock()

	if removeFromParent {
	    // 从父节点中移除自己 
		removeChild(c.Context, c)
	}
}
```

总体来看，cancel() 方法的功能就是关闭 channel：c.done；递归地取消它的所有子节点；从父节点从删除自己。达到的效果是通过关闭 channel，将取消信号传递给了它的所有子节点。goroutine 接收到取消信号的方式就是 select 语句中的读 c.done 被选中。
**timerCtx**

```go
type timerCtx struct {
    cancelCtx
	timer *time.Timer // Under cancelCtx.mu.

	deadline time.Time
}
```
