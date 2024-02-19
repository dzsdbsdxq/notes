> 题目序号：337
> 题目来源：字节跳动
> 频次：3492
>
> 解析：
> 本题主要考察是覆盖跟添加的点，个人觉得此题考察变化的意义不大，题目出的不行。

## 答案：大布丁

```go
package main

import "fmt"

func BeforeAppend(s []int) []int {
	s = append(s, 1)
	s = []int{1, 2, 3}
	return s
}

func AfterAppend(s []int) []int {
	s = []int{1, 2, 3}
	s = append(s, 1)
	return s
}

func main() {
	s := make([]int, 0)
	fmt.Println(BeforeAppend(s))
	fmt.Println(AfterAppend(s))
}
```

