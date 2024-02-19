> 题目来源：腾讯
>
> 频次：高频

## 答案：树枝

1. 一个函数中多个defer的执行顺序

   defer的作用就是把defer关键字之后的函数压入一个栈中延迟执行，多个defer的执行顺序是后进先出

   ~~~ go
   package main
   
   import "fmt"
   
   func main() {
   	defer fmt.Println("1")
   	defer fmt.Println("2")
   	defer fmt.Println("3")
   }
   // 输出
   // F:vmware_kuberneteskubernetes_kind_pro	>go run main.go
   // 3
   // 2
   // 1
   ~~~

2. defer、return、返回值的执行返回顺序

   return最先执行，先将结果写入返回值中（即赋值）；接着defer开始执行一些收尾工作；最后函数携带当前返回值退出（即返回值）。 