启动 `nginx` 。
- 停止 `nginx -s stop` 或 `nginx -s quit` 。
- 重载配置 `./sbin/nginx -s reload(平滑重启)` 或 `service nginx reload` 。
- 重载指定配置文件 `.nginx -c /usr/local/nginx/conf/nginx.conf` 。
- 查看 nginx 版本 `nginx -v` 。
- 检查配置文件是否正确 `nginx -t` 。
- 显示帮助信息 `nginx -h` 。