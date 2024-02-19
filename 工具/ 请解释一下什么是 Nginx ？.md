Nginx ，是一个 Web 服务器和反向代理服务器，用于 HTTP、HTTPS、SMTP、POP3 和 IMAP 协议。

目前使用的最多的 Web 服务器或者代理服务器，像淘宝、新浪、网易、迅雷等都在使用。

Nginx 的主要功能如下：

- 作为 http server (代替 Apache ，对 PHP 需要 FastCGI 处理器支持)

  > FastCGI：Nginx 本身不支持 PHP 等语言，但是它可以通过 FastCGI 来将请求扔给某些语言或框架处理。

- 反向代理服务器

- 实现负载均衡

- 虚拟主机