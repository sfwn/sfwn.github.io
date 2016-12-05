# docker volume use relative path

> Date: 2016-12-06
>
> Author: sfwn
>
> Title: docker volume 使用相对路径

我对 docker volume 也不是很了解，只是在 `docker run -v` 时使用到。

昨天遇到一个场景，我想要将 `/Users/sfwn/workspace/tmp` 目录挂载到容器里面作为临时输出文件夹，于是我使用下面的命令：
```shell
$ cd ~/workspace
$ docker run --rm -it --name test -v tmp:/var/output image1:0.0.1
```
这个镜像的目的是自动将 /var/output 目录下的文件打包成 rpm 格式并上传到 oss。  
执行这个命令之后，一切看起来都很正常，只是将打包好的文件上传到云上的时间却比以前都短，我没有在意。  
当我登录到 oss 之后发现文件的大小不对，明明有20多兆的文件，却只有几 KB。

当我使用将包解压后发现里面的内容是 `0.1.rpm, 0.2.rpm, 0.3.rpm`。很奇怪，我想要打包的东西却不在里面。可是当我检查宿主机的 `~/workspace/tmp` 目录时却发现没有问题。  
那么问题在哪里呢？其实，是 docker 将 **相对路径** `tmp` 解析成了 `volume` 的 name。  
可以使用 `$ docker volume ls` 和 `$ docker volume inspect tmp` 验证。  
也就是说，`tmp` 被当成了 volume 的 name 而不是相对路径了。  
其实，在 docker 文档中已经说了，[https://docs.docker.com/engine/reference/commandline/run/<Paste>](https://docs.docker.com/engine/reference/commandline/run/)，  
还有相关的 github 上的 [issue](https://github.com/docker/docker/issues/24408)。

简单解释一下，就是：
1. 使用相对路径，比如 `-v tmp:/var/output`，会将 `tmp-rpm` 解释成为 volume 的 name
2. 使用相对路径并且带 / ，比如 `cd ~` 再 `-v workspace/tmp:/var/output`，会报错，因为 volume name 只允许 [a-zA-Z0-9][a-zA-Z0-9_.-]
3. 使用绝对路径，比如 `-v ~/workspace/tmp:/var/output`，会正确地将 tmp 文件夹映射到容器里面

为什么不能使用相对路径呢，原因：docker cli 和 docker daemon 可能不在一台机器上，那么就无法解析相对路径。

Tips：
当使用 `$ docker volume inspect tmp` 时会发现它实际指向了一个物理地址 `"Mountpoint": "/var/lib/docker/volumes/tmp/_data"`。(也可以使用 `$ docker volume inspect -f {{.Mountpoint}} tmp`)
但是，当你想要查看这个路径的时候，却发现这个路径实际上并不存在，*No such file or directory*。  
其实这个不是真的在宿主机上的路径，而是在 docker vm 里面。有两种方法进入这个虚拟器查看前面的路径下的内容：
```way 1
$ docker run --rm -it -v /var/lib/docker:/docker alpine:3.4 ls /docker
```
```way 2
$ screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty
```
等我弄明白了会新开一贴细说。

当容器运行在 linux 下时，可以通过 `df` 找到容器在宿主机上的位置，例如: 
```
/var/lib/docker/devicemapper/mnt/b6e856dc57e7afbe04239028a670f186772199767763fb4c77db3768dfda415f
```
然后进如该目录的 `rootfs` 即可看到容器内部的文件了，当容器上没有 vim 或其他命令时，从宿主机确认二进制文件很方便。