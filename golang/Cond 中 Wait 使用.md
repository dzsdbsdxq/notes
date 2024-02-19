
```go
func (c *Cond) Wait() 
```

`Wait()`会自动释放 `c.L 锁`，并挂起调用者的 goroutine。之后恢复执行， `Wait()`会在返回时对 `c.L` 加锁。

除非被 Signal 或者 Broadcast 唤醒，否则 `Wait()`不会返回。

由于 `Wait()`第一次恢复时，`C.L` 并没有加锁，所以当 Wait 返回时，调用者通常 并不能假设条件为真。如下代码：。

取而代之的是, 调用者应该在循环中调用 Wait。（简单来说，只要想使用 condition，就必须加锁。）

```go
c.L.Lock() 
for !condition() {
c.Wait()
}
... make use of condition ... 
c.L.Unlock()
```