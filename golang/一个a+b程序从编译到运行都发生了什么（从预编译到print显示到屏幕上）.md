> 题目序号：（2799） 
>
> 题目来源：360
>
> 频次:  1

**答案：Zbbxd**

简要来说，整个流程如下：
**源码 --> 编译 --> 链接 --> 可执行文件 --> 执行输出**

Golang为编译型语言，需要将源代码文件编译之后才能执行。可将Golang的编译过程视为构建Golang程序的第一步。

Go程序的编译过程：文本 -> 编译 -> 二进制可执行文件
使用`go build -x hello.go`命令可以看到go源码文件编译和链接的过程
![Pasted image 20220508222620](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/Pasted image 20220508222620.png)
**编译**：文本代码 -> 目标文件`（.o， .a）`
**链接**：将目标文件合并为可执行文件

可执行文件在不同的操作系统规范不一样
以`Linux`的可执⾏⽂件`ELF`(Executable and Linkable Format) 为例

![Pasted image 20220508222934](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/Pasted image 20220508222934.png)
通过`entry point`找到 `Go进程`的执⾏⼊⼝，使⽤`readelf`。进一步找到`Go进程`要从哪里启动了
![Pasted image 20220508223016](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/Pasted image 20220508223016.png)
`CPU`⽆法理解⽂本，只能执⾏⼀条⼀条的⼆进制机器码指令，每次执⾏完⼀条指令，`pc寄存器`就指向下⼀条继续执⾏。在 64 位平台上`pc 寄存器 = rip`。
计算机会自上而下，依次执行汇编指令：
![Pasted image 20220508223136](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/Pasted image 20220508223136.png)
Go 语⾔是⼀⻔有`runtime`的语⾔
![Pasted image 20220508223324](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/Pasted image 20220508223324.png)
可以认为`runtime`是为了实现额外的功能，⽽在程序运⾏时⾃动加载/运⾏的⼀些模块。
Go语言中，`运行时`、`操作系统`和`程序员定义代码`之间的关系如下图：
![Pasted image 20220508223348](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/Pasted image 20220508223348.png)
在Go语言中，runtime主要包括：

Scheduler：调度器管理所有的 G，M，P，在后台执⾏调度循环
Netpoll：⽹络轮询负责管理⽹络 FD 相关的读写、就绪事件
Memory Management：当代码需要内存时，负责内存分配⼯作
Garbage Collector：当内存不再需要时，负责回收内存
这些模块中，最核⼼的就是 Scheduler，它负责串联所有的runtime 流程。

通过 `entry point` 找到 Go 进程的执⾏⼊⼝：
![Pasted image 20220508223016](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/Pasted image 20220508223016-16523240145061.png)
`runtime.rt0_go`的相关处理过程：

-   开始执行用户`main函数`（从这里开始进入调度循环）
-   初始化内置数据结构
-   获取CPU核心数
-   全局`m0`、`g0`初始化
-   `argc`、`argv`处理

`m0`为Go程序启动后创建的`第一个线程`