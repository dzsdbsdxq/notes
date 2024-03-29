> **题目序号：**584
>
> **题目来源**：滴滴
>
> **频次**：1

**答案1：**（重拾）

1.使用Node.js时，CPU性能或内存限制任务会变慢。Node.js基于JavaScript，一种解释型语言。解释的语言大多比编译语言慢。使用Node的动态类型特性，它不会达到Go可以实现的原始性能。相比之下，Golang的表现类似于C或C++。只有在网络通信或数据库交互的情况下，节点才能保持高性能。

2.并行和可扩展

由于Go中goroutine,Golang可扩展。Goroutines(协程)可以帮助多个线程同时执行。而且并行任务的执行是高效可靠的。由于Node.js是单线程的，指令按顺序执行，这限制了它在大规模扩展期间的能力并且同时执行大量并行处理。

3.语言成熟度

Golang的年代相当健壮且成熟，而对于Node,不断变化的API会成为编写和使用Node模块的开发人员遇到API问题的原因。当谈到开发 "商业"解决方案时，Golang是最好的选择。Golang的性能闪电般快，它的goroutine允许极好的可扩展性和并发性，并且它有助于构建更强大的应用程序。