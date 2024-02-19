> **题目序号：**（2370）
> **题目来源：** 小米
> **频次:**  1

## 答案：小强

1. 使用channel

```go
func main() {
	ch := make(chan struct{}, 10)

	for i := 0; i < 10; i++ {
		go func() {
			fmt.Println("do work...")
			ch <- struct{}{}
		}()
	}

	for i := 0; i < 10; i++ {
		<-ch
	}
	close(ch)

	fmt.Println("work finish")
}
```

2.使用sync.waitgroup ，errgroup的功能更加强大能够捕获协程中的错误。

```go
func main(){
	var wg sync.WaitGroup
	wg.Add(10)

	for i := 0; i < 10; i++ {
		go func() {
			fmt.Println("do work...")
			wg.Done()
		}()
	}

	wg.Wait()
	fmt.Println("work finish")
}
```

3.使用errgroup.Group 相比waitgroup它的功能更加强大能够捕获协程中的错误

```go
func main(){
	group := new(errgroup.Group)

	for i := 0; i < 5; i++ {
		i := i
		group.Go(func() error {
		    if i >= 3 {
				return fmt.Errorf("num can not great 2 %v", i)
			}
			fmt.Println(i)
			return nil
		})
	}

	if err := group.Wait(); err != nil {
		fmt.Println(err)
	}
	
	fmt.Println("work finish")
}
```

