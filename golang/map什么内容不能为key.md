> 题目来源：
>
> 频次：1

## 答案：engine


`map[key]value`，其中key必须是可比较的，也就是可以通过`==`和`!=`进行比较，所以可以比较的类型才能作为key，其实就是等价问go语言中哪些类型是可以比较的：

什么可以比较：bool、array、numeric（浮点数、整数等）、pointer、string、interface、channel

什么不能比较：function、slice、map