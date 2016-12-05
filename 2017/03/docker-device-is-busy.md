# docker-device-is-busy

> Date: 2017-03-13
>
> Author: sfwn
>
> Title: Docker: Device is busy 问题追踪及解决方案

#### 关键词
_device is busy, docker-compose, MountFlag=slave, --live-restore_

## 场景回顾
使用命令 `docker-compose down && docker-compose up -d` 重启 docker 服务时，有时会遇到 `Device is Busy` 错误：
```diff
ERROR: for terminus_paas-master_1  Driver devicemapper failed to remove root filesystem 8acf326b1e7c1038998ac04a6851dc4e1bacea8e6d9c927f7a092026aeb67970: Device is Busy
```
使用 `docker-compose down` 会导致一些容器异常结束，使其状态变为 _Dead_（可以用 `docker ps -a -f status=dead` 进行查看）。此时继续使用 `docker-compose up -d` 时，会报如下错误：
```diff
ERROR: for paas-master Cannot start service paas-master: Container is marked for removal and cannot be started.
ERROR: Encountered errors while bringing up the project.
```
意思是，该容器已经被做好标记需要删除（状态为 Dead），所以不能启动它。

## 问题重现
一开始对这个问题没有什么头绪。后来发现，其实这个看似偶然的问题可以被很容易的复现（reproduce），步骤如下：

1. `docker-compose up -d` 启动容器
2. 启动 **nginx**
3. 修改 **docker-compose.yml** 文件中任一镜像的版本
4. 运行 `docker-compose up -d` 更新容器

前三步都没问题。运行至第 4 步时，_docker-compose_ 尝试 ***recreate*** 第三步中改变过版本的那个容器，但却意外的报错了。没错，就是 ***Device is Busy***。此时检查该容器状态，果然已经变成了 _Dead_ 。

回想在我们的生产环境中，确实每台 PaaS Master 机器加入 PaaS 时都顺带安装了 nginx 。

### 为什么和 nginx 有关
难道 nginx 会干扰 docker 容器的起停？

> 是的。实际上，任何 systemd unit file 中带有 `PrivateTmp=true` 的 serivce 都有可能导致这个问题，比如 `ntpd.service`，`systemd-udevd service`。

首先，运行以下命令来查看 docker / host / nginx 的 mount namespace：
```diff
[root@10 ~]# systemctl status docker |grep PID
 Main PID: 980 (dockerd)
[root@10 ~]# ll /proc/980/ns/mnt
lrwxrwxrwx. 1 root root 0 Apr 23 16:32 /proc/980/ns/mnt -> mnt:[4026531840]
[root@10 ~]# ll /proc/$$/ns/mnt
lrwxrwxrwx. 1 root root 0 Apr 23 16:32 /proc/3621/ns/mnt -> mnt:[4026531840]
[root@10 ~]# systemctl status nginx |grep PID
 Main PID: 3714 (nginx)
[root@10 ~]# ll /proc/3714/ns/mnt
lrwxrwxrwx. 1 root root 0 Apr 23 16:38 /proc/3714/ns/mnt -> mnt:[4026532203]
```
可以看出，docker 运行在 `host mount namespace` 上，即 **docker** 与 **host** 共享 同一个 `mount namespace`。同时可以发现，**nginx** 在 **docker** 之后启动，并且运行在 **私有的** `mount namespace` 里。

在 nginx 的 systemd 文件中有这么一行配置：
```diff
[Service]
PrivateTmp=true
```
这行配置保证了 nginx 会运行在 **私有的** `mount namespace` 里，也正是该配置导致 docker 因为无法删除正在被其他 `mount namespace` 中的挂载点（mount point）使用的文件夹而报 `Device is busy` 的错误：

此时如果通过 `systemctl restart nginx` 重启 nginx，使用 `grep devicemapper/mnt /proc/<nginx-master-pid>/mounts` 可以发现，docker 的挂载点泄露到了 nginx 的 mount namespace 里。
```diff
[root@10 ~]# systemctl status nginx |grep PID
 Main PID: 3714 (nginx)
[root@10 ~]# grep devicemapper/mnt /proc/3714/mounts
[root@10 ~]#
[root@10 ~]# systemctl restart nginx
[root@10 ~]# systemctl status nginx |grep PID
 Main PID: 4744 (nginx)
[root@10 ~]# grep devicemapper/mnt /proc/4744/mounts
/dev/mapper/docker-253:0-67984082-1b610c14980176332f5646fabc9d809e0f4604588618f4adbe3f8f78f1736bb1 /var/lib/docker/devicemapper/mnt/1b610c14980176332f5646fabc9d809e0f4604588618f4adbe3f8f78f1736bb1 xfs rw,seclabel,relatime,nouuid,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota 0 0
......
```
因此，当 docker 需要删除一个容器时被 nginx 阻止了，因为  `/var/lib/docker/devicemapper/mnt/1b610c14980176332f5646fabc9d809e0f4604588618f4adbe3f8f78f1736bb1` 这个目录同时被 nginx 使用了，而 nginx 的 mount namespace 是私有的，docker 无法删除这个正在被 nginx 使用的文件夹，从而报错。

## 解决方法
最根本的方法是，当机器加入 PaaS 时，在 `docker.service` 中加上：
```diff
[Service]
MountFlags=slave
```
来保证 docker 及 container 运行在 **私有** 的 mount namespace 上。当 docker 以这种方式启动时，可以使用 `grep /mnt /proc/self/mountinfo` 来检测以这种方式启动的容器确实看不到 mount point 了，而之前遗留的挂载点仍然可以看到。

### 很多线上机器已经跑着业务容器不能重启 docker daemon 怎么办？
在生产中，需要尽可能保证所有的操作 **不影响正在运行的业务容器**。

所以，在已有条件下，既不能停止 nginx，又不能重启 docker daemon，唯一的方法只有在 `docker-compose down` 出错的情况下通过执行
```diff
# docker rm -f $(docker ps -a -q -f status=dead)
```
来强制删除 dead container 了。虽然仍然会有错误信息提示，但是可以看到 Dead container 确实被删除了。之后可以正常运行 `docker-compose up -d` 重启 PaaS-Master。

### 有没有方法可以在不影响业务容器的前提下重新配置 docker daemon 呢？

有。docker 从 1.12.0 开始支持在 **不重启 container** 的情况下重启 docker daemon，只需要在 docker daemon 启动时增加 `--live-restore=true` 这个参数。可惜的是，该参数直到目前仍不支持 daemon reload（类似 nginx 的 reload 和 restart），只能重启 dockerd 后生效。

## 本质
挂载点泄露实际上是 RHEL/CentOS 内核的一个 **bug**，目前预计会在 RHEL7.4 kernel 中修复。因此该问题理论上在 Fedora 中不是存在的。

目前在 RHEL/CentOS 下需要在 `docker.service` 中增加 `MountFlags=slave` 来保证 `mount namespace` 是私有的，不会造成挂载点泄露。

另外，挂载点泄露在各种 storage driver 中都存在，比如最常见的 `devicemeppaer` 和 `overlay`。

但是，目前 RHEL/CentOS 版本下 `MountFlags=slave` 和 `--live-restore` 两个给力的参数不能同时存在。因为 `MountFlags=slave` 会导致 docker daemon 每次重启时私有挂载命名空间都会发生变化，而 `--live-restore` 又相当于使得容器持有了变化之前的旧的挂载点信息，因此，当重启 docker daemon 之后，执行 `docker exec` 试图进入容器时会报错：
```diff
rpc error: code = 13 desc = invalid header field value "oci runtime error: exec failed: container_linux.go:247: starting container process caused \"process_linux.go:75: starting setns process caused \\\"fork/exec /proc/self/exe: no such file or directory\\\"\"\n"
```
值得一提的是，虽然无法进入容器，但是容器依旧工作正常并且 `docker logs` 也没问题。

## 总结

- 对于生产环境，遇到 `docker-compose up -d` 造成的 `Device is busy` 问题时，使用 `docker rm -f [container]` 后重新 `docker-compose up -d` 即可；
- 对于新的机器，当前版本下为 `docker.service` 加上 `MountFlags=slave` 即可。

参考资料

- [Driver devicemapper failed to remove root filesystem. Device is busy](https://github.com/docker/docker/issues/27381)
- [Problem with docker exec after daemon restart](https://bugzilla.redhat.com/show_bug.cgi?id=1419877)
- [The RHEL docker package does not currently support --live-restore](https://access.redhat.com/articles/2938171)
