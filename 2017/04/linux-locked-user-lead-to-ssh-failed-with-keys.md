# Linux 用户锁定导致 ssh 密钥登录失败

> Date: 2017-04-08
>
> Author: sfwn

## 梦回现场
周五的时候遇到这样一个奇怪的问题：

在一个客户的服务器上（假设为 _1.2.3.4_）新建了一个用户 aa，将我本地的 ssh 公钥拷贝到 aa 用户的 `/home/aa/.ssh/authorized_keys`，并且设置 `/home/aa/.ssh/` 目录的权限为 _700_，`/home/aa/.ssh/authorized_keys` 文件权限为 600。

检查 `/etc/ssh/sshd_config` 配置，确认以下配置正确:
```
AuthorizedKeysFile      .ssh/authorized_keys
```

然而，此时本地通过 `ssh aa@1.2.3.4` 仍然提示需要输入密码。

在尝试过程中偶然使用 `ssh-copy-id root@1.2.3.4` 将本地的 公钥 拷贝到了服务器的 **root** 用户下，然后 `ssh root@1.2.3.4` 竟然进去了。此时 aa 用户仍然提示需要输入密码。

## 模拟现场
1. 一开始使用 `ssh -vvv aa@1.2.3.4` 试图从 verbose log 中发现错误信息，然而并没有发现有价值的内容
2. 决定在服务器 _1.2.3.4_ 上的 _5555_ 端口以 `debug` 模式启动 `sshd` 进程，看看会不会有什么收获。
  - 使用命令在 5555 端口启动了：`# /usr/sbin/sshd -p 5555 -d`
  - 本地：`$ ssh aa@1.2.3.4 -p 5555`
  - 在 `debug` 模式的 `sshd` 日志中发现了问题所在：
```
User aa not allowed because account is locked
input_userauth_request: invalid user aa [preauth]
```

看到这个错误信息大家都明白是什么一回事了吧。

## 解决问题
`account is locked`：既然用户被锁住了，那就 unlock 吧。

```
# passwd -f -u aa
Unlocking password for user aa.
passwd: Success
```
然后本地运行 `ssh aa@1.2.3.4` 使用密钥连接成功。

## 分析总结
这台服务器由用户提供，有一些特殊的安全策略。因此在创建新用户之后，该用户默认是 **locked**，需要手动解锁。
