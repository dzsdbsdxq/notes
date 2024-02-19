iptables ，是一个配置 Linux 内核防火墙的命令行工具。功能非常强大，对于我们开发来说，主要掌握如何开放端口即可。例如：

- 把来源 IP 为 192.168.1.101 访问本机 80 端口的包直接拒绝：`iptables -I INPUT -s 192.168.1.101 -p tcp --dport 80 -j REJECT` 。

- 开启 80 端口，因为web对外都是这个端口

  ```
  iptables -A INPUT -p tcp --dport 80 -j ACCEP
  ```

- 另外，要注意使用 `iptables save` 命令，进行保存。否则，服务器重启后，配置的规则将丢失。