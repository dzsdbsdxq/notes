> 题目来源：BIGO

答案：树枝

**1.解题思路**

1. 我们现在有一个“有序的切片”
2. 根据这个切片将map有序输出

**2.这里写的是一个模板，根据具体的slice与map来写出代码**

~~~ go 
package main

import "fmt"

func sortMap(s []string, m map[string]string) {
	for _, k := range s {
		fmt.Println(m[k])
	}
}

func main() {
	s := []string{"k1", "k2", "k3"}
	m := map[string]string{"k2": "v2", "k1": "v1", "k3": "v3"}
	sortMap(s, m)
}
~~~

 