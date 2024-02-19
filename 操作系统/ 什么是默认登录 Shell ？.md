在 Linux 操作系统，`""/bin/bash""` 是默认登录 Shell，是在创建用户时分配的。

使用 chsh 命令可以改变默认的 Shell 。示例如下所示：

```
# chsh <用户名> -s <新shell>
# chsh linuxtechi -s /bin/sh
```

