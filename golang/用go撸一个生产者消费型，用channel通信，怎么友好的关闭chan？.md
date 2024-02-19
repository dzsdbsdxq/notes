> **题目序号：**（2804, 2982）
> **题目来源：** 七牛云、字节跳动
> **频次:**  2

## 答案：小强

如何优雅的关闭channel 记住两点

1. 向一个已关闭的channel发送数据会panic
2. 关闭一个已经关闭的channel会panic

**针对单个生产者 在发送侧关闭channel即可**

单个生产者单个消费者模型

```go
func main() {
	var ch = make(chan int)

	// 单生产者
	go func() {
		for i := 1; i < 100; i++ {
			ch <- i
		}
		close(ch)
	}()

	// 消费者
	go func() {
		for elem := range ch {
			fmt.Println(elem)
		}
	}()

	select {}
}
```

单个生产者多个消费者模型

```go
func main() {
	var ch = make(chan int)

	// 单生产者
	go func() {
		for i := 1; i < 100; i++ {
			ch <- i
		}
		close(ch)
	}()

	// 多消费者
	for i := 0; i < 100; i++ {
		go func() {
			for elem := range ch {
				fmt.Println(elem)
			}
		}()
	}

	select {}
}
```

**针对多个生产者 不应该关闭生产者，消费者通知生产者不发送数据即可**

多生产者单消费者模型

```go
func main() {
	var ch = make(chan int)
	var stopCh = make(chan struct{})
	// 多生产者
	for i := 1; i <= 100; i++ {
		go func(n int) {
			for {
				select {
				case ch <- n:
				case <-stopCh:
					return
				}
			}
		}(i)
	}

	// 单消费者
	go func() {
		for elem := range ch {
			fmt.Println(elem)
			if elem == 100{
				return
			}
		}
	}()

	select {}
}
```

多生产者多消费者模型

```go
func main() {
	var ch = make(chan int)
	// 停止信号
	var stopCh = make(chan struct{})
	// 协调者
	var toStopCh = make(chan struct{}, 1)
	// 多生产者
	for i := 1; i <= 100; i++ {
		go func(n int) {
			for {
				select {
				case ch <- n:
				case <-stopCh:
					return
				}
			}
		}(i)
	}

	// 接收通知给协调者
	go func() {
		<-toStopCh
		close(stopCh)
	}()

	// 多消费者
	for i := 0; i < 100; i++ {
		go func() {
			for  {
				select {
				case elem := <-ch:
					if elem == 100 {
						toStopCh <- struct{}{}
						return
					}
					fmt.Println(elem)
				}
			}
		}()
	}

	select {}
}
```


#### 