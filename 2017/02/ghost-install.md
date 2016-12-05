# ghost install

> Date: 2017-02-17
>
> Author: sfwn
>
> Title: Ghost 安装配置记录

搬瓦工 vps 的内存不够，安装 ghost 的时候虽然用了 ulimt 和 cpulimt 命令来限制 cpu 和内存，但 node 进程还是被 kill 掉。最后嫌麻烦在本地用 docker 启动 ghost，在 vps 上用 nginx 反向代理以及本机 ssh 远程端口转发简易实现 ghost blog 搭建。

下面记录遇到的问题：

- docker 启动命令

```
$ docker run --name ghost --restart=always -d -p 2368:2368 -v ~/ws/ghost/:/var/lib/ghost ghost:latest
```
 ghost 默认启动的是 development 环境。

- 如何切换到 production 环境 ？

 docker 启动命令带上 `-e NODE_ENV=production` 。
注意，此时会启动失败，需要在 config.js 中的 production 环境下添加以下内容：

```
paths: {
  contentPath: path.join(process.env.GHOST_CONTENT, '/')
}
```

 该内容与 dev 环境中内容相同，只不过 prod 的配置中丢了。

- 切换到 prod 环境后 ghost 数据丢了 ？

 ghost 容器启动时分明把数据持久化了。那为什么之前写的博客不见了，博客样式都变了？其实东西都在，需要指定测试环境数据库 `/data/ghost-dev.db` 。

- 无法发送邀请邮件？

 之前没有搭建和配置邮件服务器的经验，所以在这里栽了大跟头。实际上不需要搭建自己的邮件服务器，即使想搭建，也有免费的企业域名邮件服务器可以选择，比如 [网易企业邮](ym.163.com) 和 [腾讯企业邮箱](exmail.qq.com)。不过我的域名 sfwn.online 腾讯企业邮不识别，所以只能选择网易企业邮，并设置好 MX 记录。

 实际上没有必要使用企业邮箱，ghost 使用的是 [Nodemailer](https://nodemailer.com/about/)。所以只要是配置正确，所有邮箱应该都能发送邀请邮件，qq 邮箱当然也是支持的。

 目前我的邮箱配置如下（使用的是网易免费企业域名邮箱）：
```
mail: {
    transport: 'SMTP',
    from: '林 俊 👻 <sfwn@sfwn.online>',
    options: {
        host: 'smtp.ym.163.com',
        port: 25,
        auth: {
            user: 'sfwn@sfwn.online',
            pass: '********'
            }
        }
    },
```

- 本地编辑，服务端同步

 在本地写文章，`crontab` 每五分钟 rsync 到服务器端，保持同步。

- rsync 同步到服务器后，后台 `/ghost` 无法登录，会报 `SQLITE_READONLY: attempt to write a readonly database` 错误

 1. 注意 selinux 的策略，需要关闭或者临时关闭: `setenforce 0`
 2. `/path/to/ghost/data/` 这个文件夹需要 `+w`
 3. `/data/xx.db` 需要 `+w`
 4. 可能需要 `docker restart ghost`

- 需要看看 rsync 有没有什么选项可以设置 mode，暂时是把本地开了 `u+w`

 `#*/5 * * * * rsync -av --delete --chown=root:root /Users/sfwn/ws/ghost/ gce:/root/ghost`
 注意：需要 rsync 3.1.0 以上版本，目前 rsync 最新版是 3.1.2，osx 上使用 homebrew 升级，yum 上使用:
```shell
$ wget http://dl.fedoraproject.org/pub/fedora/linux/releases/24/Everything/x86_64/os/Packages/r/rsync-3.1.2-2.fc24.x86_64.rpm
$ rpm -Uvh rsync-3.1.2-2.fc24.x86_64.rpm
```

- 点击左上角头像时会跳转到 `127.0.0.1:2368`

 启动 docker 命令时忘了加上 `-e NODE_ENV=production` 了，而 development 配置的 url 是 127.0.0.1:2368