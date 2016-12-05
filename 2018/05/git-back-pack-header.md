# git: fatal: protocol error: bad pack header

> Date:  2018-05-19
>
> Author: sfwn

```bash
$ git clone http://${REPO_URL}/${ROGANIZATION}/${PROJECT}
Cloning into '${PROJECT}'...
remote: Counting objects: 37797, done.
remote: aborting due to possible repository corruption on the remote side.
fatal: protocol error: bad pack header
git clone   0.04s user 0.04s system 0% cpu 8.823 total
```

这是因为服务端在内存里做压缩时 oom 了。

https://stackoverflow.com/a/28263927

在服务端设置 `git config --global pack.window "0"` 可以不用内存做压缩。

如果服务端使用的是 smart http protocol，那么全局设置不生效，需要进入具体 git repo 目录设置 `git config pack.window "0"`。