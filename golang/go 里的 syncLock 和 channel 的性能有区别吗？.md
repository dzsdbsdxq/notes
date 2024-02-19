> **题目序号：**（2364）
> **题目来源：** 小米
> **频次:**  1

## 答案：小强

channel的底层也是用了syns.Mutex,算是对锁的封装，性能应该是有损耗的，用测试的数据更有说服力

```go
package channel

import "sync"
var mutex = sync.Mutex{}
var ch = make(chan struct{}, 1)
func UseMutex() {
	mutex.Lock()
	mutex.Unlock()
}
func UseChan() {
	ch <- struct{}{}
	<-ch
}
```

```go
package channel

import "testing"

func BenchmarkUseMutex(b *testing.B) {
	for i := 0; i < b.N; i++ {
		UseMutex()
	}
}

func BenchmarkUseChan(b *testing.B) {
	for i := 0; i < b.N; i++ {
		UseChan()
	}
}
```

执行bench命令

```go
go test -bench=.
```

测试结果如下

```go
BenchmarkUseMutex-8     87120927                13.61 ns/op
BenchmarkUseChan-8      42295345                26.47 ns/op
PASS
ok      mytest/channel  2.906s
```

根据压测结果来说Mutex 比 channel的性能快了两倍左右