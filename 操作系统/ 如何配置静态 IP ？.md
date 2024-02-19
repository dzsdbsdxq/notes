```css
测试服务器OS: Centos 6.5 x64
本机OS: Ubuntu 14.04 x64
```

由于Virtualbox当时安装Centos 6.5的时候设置的是自动获取的IP，所以局域网内每次启动，IP有时候会变化
 如果本地测试最好固定静态IP



```bash
#用户root登陆到服务器，编辑配置信息
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

设置如下内容，有则改之，无则添加



```cpp
DEVICE=eth0
BOOTPROT=static
IPADDR=192.168.1.200
GATEWAY=192.168.1.1
NETMASK=255.255.255.0
ONBOOT=yes
```

之后保存
 然后重启网卡即可



```undefined
service network restart
```

现在试试吧，IP已经变为192.168.1.200
 直接远程登陆



```css
ssh root@192.168.1.200
```



