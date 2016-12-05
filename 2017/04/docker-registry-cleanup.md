# docker registry cleanup

> Date: 2017-04-01
>
> Author: sfwn

在搭建私有 docker registry 之后，早晚会遇到磁盘空间占用过大的问题。docker registry 从 `2.4` 版本开始可以利用 docker registry 的 Restful api 来清理过时的 docker images 释放磁盘空间。

> 本文涉及到的脚本和文档都放在 [Gist] [0] 上了。

#### Docker Registry V2 API
用到的 docker registry api 在 [registry api v2](https://docs.docker.com/registry/spec/api/#detail) 中可以找到，其中涉及到的主要 api 如下：

1. `GET /v2/_catalog` 列出所有 repository [[ref]] [1]
2. `GET /v2/<name>/tags/list` 列出当前 repository 下的所有 image tags [[ref]] [2]
3. `DELETE /v2/<name>/manifests/<reference>` 指定将会被删除的 image tag [[ref]] [3]

在我的业务场景中，每次执行 [清理脚本] [4] 最多只会保留最近的 5 个 image tag，其他历史 tag 均会被清理。

#### 具体步骤
##### 1. 前提
安装 `jq` 命令行工具用来解析 json 。
##### 2. 步骤
- 修改 registry 容器内的配置文件 `/etc/docker/registry/config.yml` 以支持 registry 清理工作
```
# 可使用 docker cp
storage:
  delete:
    enabled: true
```
- 重新启动容器
- 下载并运行 [清理脚本] [4] (可以使用 `tee` 将输出重定向到标准输出和 log 文件)，可以看见执行过程
- 在宿主机上执行 `docker exec -it {registry-id} /bin/registry garbage-collect /etc/docker/registry/config.yml` 调用 registry#gc 来真正执行清理动作 (清理上步中标记为需要清理的 docker image)。这一步比较耗时，由需要清理的镜像数量而定，可能需要等待一段时间后才会开始打印日志，耐心等待即可。
> note: 在 cron 中执行该命令时，需要去掉 `-t` 参数，否则会报错：`the input device is not a TTY`

- `docker stop {registry-id} && docker start {registry-id} (直接 `docker restart {registry-id} 可能会造成无法进入 registry 容器的 issue)
- 修改配置文件为 `storage: delete: enabled: false`

以上就完成了 docker registry 的一次清理。

因为懒，当然不会满足每次手动执行任务了，所以可以使用 cron 定时任务来帮助我们自动清理 registry。

cron 任务如下，每天凌晨 3 点开始清理：
`0 3 * * * bash /root/docker-reg-gc/cron-docker-reg-gc.sh 2>&1 |tee /root/docker-reg-gc/log/$(date +\%Y-\%m-\%d).log`

当定时任务执行失败时，会自动发送邮件通知你，需要手动解决问题了 :(

如果使用 cron 出现问题，请移步我的 [Gist] [0] :)

[0]: https://gist.github.com/sfwn/7453e78be0374b3d53f1e44f5bb8beef
[1]: https://docs.docker.com/registry/spec/api/#listing-repositories
[2]: https://docs.docker.com/registry/spec/api/#listing-image-tags
[3]: https://docs.docker.com/registry/spec/api/#deleting-an-image
[4]: https://gist.github.com/sfwn/7453e78be0374b3d53f1e44f5bb8beef#file-1-docker-reg-gc-sh