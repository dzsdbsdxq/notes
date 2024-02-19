> **题目序号：**(3044)
>
> **题目来源：**小米
>
> **频次：**1

## **答案1：**(dema)

```go
package main

import (
  "fmt"
  "runtime"
  "sync"
  "time"
)

// Task 任务接口
type Task interface {
  Execute()
}

// Pool 协程池
type Pool struct {
  TaskChannel chan Task // 任务队列
}

// NewPool 创建一个协程池
func NewPool(cap ...int) *Pool {
  // 获取 worker 数量
  var n int
  if len(cap) > 0 {
    n = cap[0]
  }
  if n == 0 {
    n = runtime.NumCPU()
  }

  p := &Pool{
    TaskChannel: make(chan Task),
  }

  // 创建指定数量 worker 从任务队列取出任务执行
  for i := 0; i < n; i++ {
    go func() {
      for task := range p.TaskChannel {
        task.Execute()
      }
    }()
  }
  return p
}

// Submit 提交任务
func (p *Pool) Submit(t Task) {
  p.TaskChannel <- t
}

// EatFood 吃饭任务
type EatFood struct {
  wg *sync.WaitGroup
}

func (e *EatFood) Execute() {
  defer e.wg.Done()
  fmt.Println("eat cost 3 seconds")
  time.Sleep(3 * time.Second)
}

// WashFeet 洗脚任务
type WashFeet struct {
  wg *sync.WaitGroup
}

func (w *WashFeet) Execute() {
  defer w.wg.Done()
  fmt.Println("wash feet cost 3 seconds")
  time.Sleep(3 * time.Second)
}

// WatchTV 看电视任务
type WatchTV struct {
  wg *sync.WaitGroup
}

func (w *WatchTV) Execute() {
  defer w.wg.Done()
  fmt.Println("watch tv cost 3 seconds")
  time.Sleep(3 * time.Second)
}

func main() {
  p := NewPool()
  var wg sync.WaitGroup
  wg.Add(3)
  task1 := &EatFood{
    wg: &wg,
  }
  task2 := &WashFeet{
    wg: &wg,
  }
  task3 := &WatchTV{
    wg: &wg,
  }
  p.Submit(task1)
  p.Submit(task2)
  p.Submit(task3)
  // 等待所有任务执行完成
  wg.Wait()
}
```

