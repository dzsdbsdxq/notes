> **题目序号：**(16,17)
> **题目来源：**深信服
> **频次：**2

**答案1：**（阿纪、）

1.假如在协程中打印for的下标i或当前下标的元素,会随机打印载体中的元素.
    原因有二: 

1. golang是值拷贝传递
   for循环很快就执行完了，但是创建的10个协程需要做初始化。上下文准备，堆栈，和内核态的线程映射关系的工作，是需要时间的，比for慢，等都准备好了的时候，会同时访问i。这个时候的i肯定是for执行完成后的下标。（也可能有个别的协程已经准备好了，取i的时候，正好是5，或者7，就输出了这些数字）。

   解决的方法就是闭包，给匿名函数增加入参，因为是值传递，所以每次for创建一个协程的时候，会拷贝一份i传到这个协程里面去。
   或者在开启协程之前声明一个新的变量 = i。

2.假如当前for是并发读取文件
    程序会panic:too many open files
    解决的方法:通过带缓冲的channel和sync.waitgroup控制协程并发量。