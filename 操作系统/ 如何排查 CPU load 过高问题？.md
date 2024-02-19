###### 1. 首先排查哪些进程cpu占用率高。 通过命令 ps ux

[![image](https://images2015.cnblogs.com/blog/731331/201608/731331-20160817133313328-137128375.png)](http://images2015.cnblogs.com/blog/731331/201608/731331-20160817133312687-385288873.png)

###### 2. 查看对应java进程的每个线程的CPU占用率。通过命令：ps -Lp 15047 cu

[![image](https://images2015.cnblogs.com/blog/731331/201608/731331-20160817133314484-415014403.png)](http://images2015.cnblogs.com/blog/731331/201608/731331-20160817133313875-637400485.png)

###### 3. 追踪线程内部，查看load过高原因。通过命令：jstack 15047。

或者打印线程 jstack `pidof java` > stack.out

查找到对应的threadid, 再反查代码。