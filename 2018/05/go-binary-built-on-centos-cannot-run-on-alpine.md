# go binary built on centos cannot run on alpine

> Date:  2018-05-23
>
> Author: sfwn

打包基础镜像：centos7
运行时基础镜像：alpine3.7

运行时异常：`ash: collector: not found`

因为默认会使用动态链接，alpine 使用的是 musl libc，centos 上是 glibc，go binary 启动时找不到 glibc 而报错。


解决方法：
1. `CGO_ENABLED=0` 禁止使用动态链接库
2. 打包基础镜像和运行时基础镜像一致

使用 `ldd` 可以查看 go binary 是否使用了动态链接库：
```bash
[sfwn@me ~]$ sudo docker run --rm -ti localhost:5000/9d7749b1c8e6d8e536d8fd40833699a81527059387287:latest ldd bin/collector
ldd: bin/collector: Not a valid dynamic program
[sfwn@me ~]$ sudo docker run --rm -ti localhost:5000/9d7749b1c8e6d8e536d8fd40833699a81527057563951:latest ldd bin/collector
linux-vdso.so.1 =>  (0x00007ffdd24bf000)
libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f5675db6000)
libc.so.6 => /lib64/libc.so.6 (0x00007f56759f3000)
/lib64/ld-linux-x86-64.so.2 (0x00007f5675fd6000)
```


参考链接：
- https://stackoverflow.com/questions/36279253/go-compiled-binary-wont-run-in-an-alpine-docker-container-on-ubuntu-host
- https://stackoverflow.com/questions/34729748/installed-go-binary-not-found-in-path-on-alpine-linux-docker/35613430#35613430