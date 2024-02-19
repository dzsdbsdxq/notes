> 题目序号：226
>
> 题目来源: 映客 
>
> 频次 1

**答案1：**（苦痛律动）

go语言在执行goroutine的时候、是没有返回值的、这时候我们要用到go语言中特色的channel来获取返回值。
通过channel拿到返回值有两种处理方式，一种形式是具有go风格特色的，即发送给一个for channel 或 select channel 的独立goroutine中，由该独立的goroutine来处理函数的返回值。还有一种传统的做法，就是将所有goroutine的返回值都集中到当前函数，然后统一返回给调用函数。

使用channel比较符合go风格，举个一个例子

**使用自定义channel类型**

```go
package main

import (
	"fmt"
	"strings"
)

type Result struct {
	allCaps string
	length  int
}

func capsAndLen(words []string, c chan Result) {
	defer close(c)
	for _, word := range words {
		res := new(Result)
		res.allCaps = strings.ToUpper(word)
		res.length = len(word)
		c <- *res // 写 channel
	}
}

func main() {
	words := []string{"lorem", "ipsum", "dolor", "sit", "amet"}
	c := make(chan Result)
	go capsAndLen(words, c) // 启动goroutine
	for res := range c {
		fmt.Println(res.allCaps, ",", res.length)
	}
}

```

**参考资料**

https://stackoverflow.com/questions/17825857/how-to-make-a-channel-that-receive-multiple-return-values-from-a-goroutine