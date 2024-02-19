> 题目序号：（4426，4451） 
>
> 题目来源：知乎
>
> 频次: 2

**答案：**Zbbxd

1. 值传递只会把参数的值复制⼀份放进对应的函数，两个变量的地址不同，不可相互修改。  
2. 地址传递(引⽤传递)会将变量本身传⼊对应的函数，在函数中可以对该变量进⾏值内容的修改。
3. golang默认都是采用值传递，即拷贝传递，有些值天生就是指针（slice、map、channel）
   举例：

```go
package main

import (
    "fmt"
)

func main() {
    // map
    m := make(map[int]string)
    m[0] = "a"
    m[1] = "b"
    changeMap(m)
    fmt.Printf("map:%+v", m)  //输出 map:map[0:aaa 1:b]
    fmt.Println()

    //array
    var a = [2]string{"a", "b"}
    changeArray(a)
    fmt.Printf("array:%+v", a) //输出array:[a b]
    fmt.Println()

     //slice
    var s = []string{"a", "b"}
    changeSlice(s)
    fmt.Printf("slice:%+v", s) //输出slice:[aaa b]
}

func changeMap(m map[int]string) {
    m[0] = "aaa"
}

func changeArray(a [2]string) {
    a[0] = "aaa"
}

func changeSlice(s []string) {
    s[0] = "aaa"
}
```

可以看出来map和slice都是指针传递，即函数内部是可以改变参数的值的。而array是数组传递，不管函数内部如何改变参数，都是改变的拷贝值，并未对原值进行处理。