> 题目序号：(370)
>
> 题目来源：
>
> 频次：1

**答案1**：(dema)

进入Go语言中(当前为1.17版本)的sleep.go文件查看源码

sleep的定义如下

```go
// Sleep pauses the current goroutine for at least the duration d.
// A negative or zero duration causes Sleep to return immediately.
func Sleep(d Duration)

// 翻译之后
// Sleep将暂停当前goroutine至少一段时间。
// 负持续时间或零持续时间会导致睡眠立即恢复。
func Sleep(d Duration)
```

Duration 的定义如下

```go
type Duration int64

const (
	minDuration Duration = -1 << 63
	maxDuration Duration = 1<<63 - 1
)
```

Go语言中的time.Sleep实现过程如下

在执行time.Sleep()时，程序会自动使用NewTimer方法创建一个新的Timer，在初始化的过程中我们会u传入当前Goroutine应该被唤醒的时间以及唤醒时需要调用的函数goroutineReady，随后会调用goparkunlock将当前Goroutine陷入休眠状态，当定时器到期时也会调用goroutineReady方法唤醒当前的Goroutine

Timer的结构和NewTimer方法源码如下

```go
type Timer struct {
	C <-chan Time
	r runtimeTimer
}

func NewTimer(d Duration) *Timer {
	c := make(chan Time, 1)
	t := &Timer{
		C: c,
		r: runtimeTimer{
			when: when(d),
			f:    sendTime,
			arg:  c,
		},
	}
	startTimer(&t.r)
	return t
}
```

[[参考链接](http://c.biancheng.net/view/5723.html)](