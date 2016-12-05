# ssh 远程端口转发 + nginx 反向代理 完成 ghost 博客访问

> Date: 2017-02-15
>
> Author: sfwn
>
> Title: ssh 远程端口转发 + nginx 反向代理 完成 ghost 博客访问

> 推荐阅读：http://blog.creke.net/722.html

- 前提：ss 服务器一台，mac 一台（无公网 ip）
- 目标：想要通过访问 ss 服务器的 2368 端口来访问实际上位于 mac 上 2368 端口 的 ghost 博客

#### 做法：
- nginx 配置
    1. 一个需要注意的地方：用 cat 来保存 ghost.conf 时，$var 都变成取值了，导致 ghost 配置文件异常
    2. 代码如下：
```nginx
upstream ghost_server {
    server localhost:2368;
}

server {
    listen 80;
    server_name sfwn.online;

    # individual nginx logs for this web vhost
    access_log  /var/log/nginx/ghost-access.log;
    error_log   /var/log/nginx/ghost-error.log;

    location / {
        proxy_pass http://ghost_server;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        Host $http_host;
    }
}
```
- ssh 端口转发
    1. 一开始在思考到底是本地端口转发还是远程端口转发，因为只有 mac 可以连接 ss 服务器，而需求是 ss 服务器的 2368 端口转到 mac 上的 2368 端口。实际上是需要在 mac 和 ss 服务器之间建立一条 ssh 隧道。如果从服务器上建立隧道，需要服务器可以访问 mac，万一服务器被攻破，那么 mac 也就被攻破了，安全性不高，弃用。只能从 mac 端发起隧道连接。显然本地端口转发不对，所以只剩下远程端口转发了。
    2. mac 使用命令建立远程端口转发（让服务器访问 mac）：`ssh -N -f -R 2368:127.0.0.1:2368 ss` ，含义是 ss 的 2368 端口转发到 mac 本机的 2368 端口

#### 缺点
只是临时转发，ssh 断了之后就 502 了，因此添加了一个定时任务，每 5 分钟执行一次。mac 下 `crontab -e` 添加任务时会报 `crontab: temp file must be edited in place` 的异常，参考下面文档解决：http://calebthompson.io/crontab-and-vim-sitting-in-a-tree/

```crontab -e
$ */5 * * * * ssh -N -f -R 2368:127.0.0.1:2368 ss
$ */30 * * * * ps -ef |grep "ssh -N -f -R" |grep -v grep |awk '{print $2}' |xargs -I {} kill -9 {}
```

#### 总结：
1. nginx 配置容易写错，谨慎
2. ssh 本地转发和远程转发刚开始使用比较容易混淆
3. 坚信想法没问题，一定可以实现

#### 后续
1. 避免单点，利用 nginx 负载均衡做一下高可用，准备在家里另一台笔记本上部署一个实例