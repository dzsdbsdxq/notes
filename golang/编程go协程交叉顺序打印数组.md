>  题目序号：(2744、97、1105...)
>
>  **题目来源**：字节  
>
>  **频次**：8

## 答案：peace

使用channel实现交叉打印0-100中的奇偶数。代码如下：

```go
package main

import (
	"fmt"
	"sync"
)

var (
	toOdd  = make(chan struct{})
	toEven = make(chan struct{})
	wg     = sync.WaitGroup{}
)

func main() {
	wg.Add(2)
	go printOdd()
	go printEven()
	wg.Wait()
	fmt.Println("-----done-----")
}

func printOdd() {
	defer wg.Done()
	for i := 1; i <= 100; i += 2 {
		if i != 1 {
			<-toOdd
		}

		fmt.Println(i)

		toEven <- struct{}{}
	}
}

func printEven() {
	defer wg.Done()
	for i := 2; i <= 100; i += 2 {
		<-toEven

		fmt.Println(i)

		if i != 100 {
			toOdd <- struct{}{}
		}
	}
}
```

