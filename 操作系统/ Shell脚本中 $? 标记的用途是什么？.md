在写一个 Shell 脚本时，如果你想要检查前一命令是否执行成功，在 `if` 条件中使用 `$?` 可以来检查前一命令的结束状态。

- 如果结束状态是 0 ，说明前一个命令执行成功。例如：

  ```
  root@localhost:~# ls /usr/bin/shar
  /usr/bin/shar
  root@localhost:~# echo $?
  0
  ```

- 如果结束状态不是0，说明命令执行失败。例如：

  ```
  root@localhost:~# ls /usr/bin/share
  ls: cannot access /usr/bin/share: No such file or directory
  root@localhost:~# echo $?
  2
  ```