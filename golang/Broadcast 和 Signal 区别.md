```go
func (c *Cond) Broadcast() Broadcast 
```

会唤醒所有等待 c 的 goroutine。

调用 Broadcast 的时候，可以加锁，也可以不加锁。

```go
func (c *Cond) Signal() 
```

Signal 只唤醒 1 个等待 c 的 goroutine。

调用 Signal 的时候，可以加锁，也可以不加锁。