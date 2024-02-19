> 题目来源：华为

解析：大布丁

n 个并发量，并发处理数组，处理完后放回数组内，使用到sync 包中的 WaitGroup 与 mutex 进行控制，假设 n 为 10，处理 长度为 20 的 int[] 类型数组，代码如下

```go
package main

import (
	"fmt"
	"sync"
)

var wg sync.WaitGroup
var m sync.Mutex

func main() {
	a := make([]int, 20)

	for i := 0; i < 20; i++ {
		a[i] = i
	}
	wg.Add(10)
	fmt.Println(a)
	for i := 0; i < 20; i++ {
		go add(a, i)
	}
	wg.Done()
	fmt.Println(a)
}

func add(s []int, pos int) {
	m.Lock()
	defer m.Unlock()
	s[pos]++
}
```
