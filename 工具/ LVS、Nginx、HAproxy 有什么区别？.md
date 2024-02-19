LVS ：是基于四层的转发。

- HAproxy ： 是基于四层和七层的转发，是专业的代理服务器。

- Nginx ：是 WEB 服务器，缓存服务器，又是反向代理服务器，可以做七层的转发。

  > Nginx 引入 [TCP 插件](https://docs.nginx.com/nginx/admin-guide/load-balancer/tcp-udp-load-balancer/)之后，也可以支持四层的转发。

🚀 区别

LVS 由于是基于四层的转发所以只能做端口的转发，而基于 URL 的、基于目录的这种转发 LVS 就做不了。

🚀 工作选择：

HAproxy 和 Nginx 由于可以做七层的转发，所以 URL 和目录的转发都可以做，在很大并发量的时候我们就要选择 LVS ，像中小型公司的话并发量没那么大选择 HAproxy 或者 Nginx 足已。

由于 HAproxy 由是专业的代理服务器配置简单，所以中小型企业推荐使用HAproxy 。

> 有些使用，使用 HAproxy 还是 Nginx ，也和公司运维对哪个技术栈的掌控程度。掌控 OK ，选择 Nginx 会更加不错。
>
> 另外，LVS + Nginx 和 LVS + HAProxy 也是比较常见的选型组合。

更多更详细的对比，感兴趣的胖友，可以看看 [《LVS、HAProxy、Nginx 负载均衡的比较分析》](https://blog.csdn.net/gzh0222/article/details/8540604) 。