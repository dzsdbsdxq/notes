> **题目序号：**（3635）
> **题目来源：** 百度
> **频次:**  1

答案：小强

使用channel+waitgroup协调并发的调度

```go
func main(){
    ch := make(chan int)
	var wg sync.WaitGroup
	wg.Add(2)
	go func() {
		defer wg.Done()
		for i := 1; i < 10; i++ {
			ch <- i
		}
		close(ch)
	}()

	go func() {
		defer wg.Done()
		for v := range ch {
			fmt.Println(v)
		}
	}()

	wg.Wait()
}
```

