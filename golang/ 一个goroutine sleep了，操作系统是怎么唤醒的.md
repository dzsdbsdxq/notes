> **题目序号：**(270)
> **题目来源：** 360
> **频次:** 2

## 答案：阿纪、

1. **goroutine唤醒**
   goroutine的唤醒涉及到一个很重要的函数(goready),它的作用就是唤醒waiting状态的goroutine.
   通过systemstack切到g0栈，在g0栈上发起调度.
   获取goroutine的状态.
   将waiting状态的goroutine切换到runable状态
   尝试唤起一个p来执行当前goroutine

2. 注释: go程序中，每个M都会绑定一个叫g0的初代goroutine，它在M的创建的时候创建，g0的主要工作就是goroutine的调度、垃圾回收等.