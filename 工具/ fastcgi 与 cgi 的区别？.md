1）cgi

> web 服务器会根据请求的内容，然后会 fork 一个新进程来运行外部 c 程序（或 perl 脚本…）， 这个进程会把处理完的数据返回给 web 服务器，最后 web 服务器把内容发送给用户，刚才 fork 的进程也随之退出。
>
> 如果下次用户还请求改动态脚本，那么 web 服务器又再次 fork 一个新进程，周而复始的进行。

2）fastcgi

> web 服务器收到一个请求时，他不会重新 fork 一个进程（因为这个进程在 web 服务器启动时就开启了，而且不会退出），web 服务器直接把内容传递给这个进程（进程间通信，但 fastcgi 使用了别的方式，tcp 方式通信），这个进程收到请求后进行处理，把结果返回给 web 服务器，最后自己接着等待下一个请求的到来，而不是退出。

🚀 综上，差别在于是否重复 fork 进程，处理请求。

关于 Nginx 和 fastcgi 是如何运行的，可以看看 [《Nginx + FastCGI运行原理》](https://www.linuxba.com/archives/7682) 。

同样，关于 cgi 可能也会存在 [《php 面试题五之 nginx 如何调用 php 和 php-fpm 的作用和工作原理》](https://phperzh.com/articles/7297) 问题。