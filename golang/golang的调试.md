> 题目来源：度小满
>
> 频次：1
>
> 整理人：lws

1、使用IDE进行（如：goland）进行debug调试，不详细说明。
2、使用golang调试工具进行调试，如：dlv（类似C语言的GDB）

**go dlv基本命令介绍**

```go
dlv attach  $PID  	## 后面的进程的ID  跟踪正在执行的go程序，查看进程内部信息状态
dlv debug main/hundredwar.go 	## 先编译，后启动调试 
dlv exec ./HundredsServer  		## 直接启动调试
dlv exec ./HundredsServer -- -port 8888 -c /home/config.xml  ## 后面加参数启动调试
dlv help 	## 查看其它dlv命令
```

