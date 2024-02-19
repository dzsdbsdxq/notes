> 题目来源: 北京合链 
>
> 频次 1

答案：苦痛律动

应该是一个for循环中作用域的问题

```go
src := []int{1, 2, 3, 4, 5}
var dst2 []*inv
for _, v := range src {
    dst2 = append(dst2, &v)
    // fmt.println(&v)
}

for _, p := range dst2 {
    fmt.Print(*p)
}
// 输出
// 5555
```

为什么呢, 因为 for-range 中 循环变量的作用域的规则限制
假如取消append()后一行的注释，可以发现循环中v的变量内存地址是一样的，也可以解释为for range相当于

```go
var i int
for j := 0; j < len(src); j++ {
    i = src[j]
    dst2 = append(dst2, &i)
}
```

而不是我们想象中的

```go
for j := 0; j < len(src); j++ {
    dst2 = append(dst2, &src[j])
}
```

如果要在for range中实现，我们可以改写为

```go
src := []int{1, 2, 3, 4, 5}
var dst2 []*int
for _, v := range src {
    new_v := v
    dst2 = append(dst2, &new_v)
    // fmt.println(&new_v)
}

for _, p := range dst2 {
    fmt.Print(*p)
}
```

**参考资料**

https://studygolang.com/articles/22495