  1、添加普通用户登陆，禁止 root 用户登陆，更改 SSH 端口号。
- 2、服务器使用密钥登陆，禁止密码登陆。
- 3、开启防火墙，关闭 SElinux ，根据业务需求设置相应的防火墙规则。
- 4、装 fail2ban 这种防止 SSH 暴力破击的软件。
- 5、设置只允许公司办公网出口 IP 能登陆服务器(看公司实际需要)
- 6、修改历史命令记录的条数为 10 条。
- 7、只允许有需要的服务器可以访问外网，其它全部禁止。
- 8、做好软件层面的防护。
  - 8.1 设置 nginx_waf 模块防止 SQL 注入。
  - 8.2 把 Web 服务使用 www 用户启动，更改网站目录的所有者和所属组为 www 。