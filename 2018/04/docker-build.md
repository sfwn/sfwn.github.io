# 如何在 docker build 中用好缓存 -- docker build 的探索

> Date:  2018-04-12
>
> Author: sfwn

本文以 从源码开始制作一个 Java 镜像说开来。

最早期的时候，只有一台机器用于打包。我们从宿主机上裸跑命令:

```
1. git clone && git checkout
2. mvn clean package -DskipTests
3. cp xxx/target/xxx/xxx.jar /app/app.jar
4. docker build image (put app into image)
5. docker push image
```

这种方式需要宿主机上装好各种打包工具，比如 maven。

但随着企业规模和业务规模逐渐增长，单台打包机成为打包瓶颈，机器资源吃紧，各项目打包任务进入队列并堆积，打包效率低下。

企业上云。在分布式环境中，如何充分利用机器闲置资源进行打包任务调度，不是难题，即分布式任务(job)调度。
我们目前采用 mesos + chronos framework 的方式。

一切 Docker 化，打包全流程也从宿主机移到了 Docker 容器中。

在 Docker 版本 17.05 之前，官方不支持 Multi-Stage，所以为了使最终构建的镜像体积最小，通常采用 Shell 脚本串联两个 Dockerfile 的方式，第一个 Dockerfile 用于从源码制作出 app.jar，第二个 Dockerfile 将 app.jar 与必要的运行时环境放在一起，制作出 Docker Image。这种方式有两个缺点，一是需要专门维护 Shell 脚本进行串联，二是中间产物(app.jar)需要显式与文件系统进行交互，并在镜像制作完成之后清理中间产物。

在 Docker 17.05 及之后版本，随着 Multi-Stage 的加入，可以将多个 docker build stage(多个 Dockerfile) 合并成在一个 Dockerfile 中，通过 `COPY --from` 的方式直接在 Stage 之间交换文件。免除了多余 Shell 脚本的维护，免除了中间产物对文件系统的潜在污染。

打包过程中耗时最长的当属依赖的下载。
在单台宿主机打包时代，Java 依赖能够被缓存在 $HOME/.m2/repository 下，多次 Docker Image 制作过程共享该依赖目录。
但在分布式云环境中，由于打包任务会被随机调度到空闲机器上，多台打包机需要共享依赖目录。

这里我们也经历了三个阶段。

第一阶段，打包提速完全依赖 docker 自身的 image layer cache，两次打包如果 content 相同，Dockerfile 相同，则打包几乎秒级完成。但是打包的一个潜在前提就是代码发生了变化，而代码发生变化，导致 content 就不同，所以 docker 自身的这一层 layer cache 实际上很难使用到。
之后便考虑到将依赖下载和编译分开，即 `mvn compile dependency:go-offline` 和 `mvn clean package` 分开。这样，代码本身发生变化，而依赖未发生变化时，依赖部分能被缓存住，提升了打包速度。但是由于 SNAPSHOT 的概念，很多时候依赖文件本身 pom.xml 没有发生变化，但是实际上依赖的 jar 包发生了变化，此时需要强制下载依赖，使缓存失效。在 Java 打包中这种情况尤为明显。

第二阶段，我们开始思考，依赖是会极易变化的，因此像第一阶段那样将依赖 cache 住，能复用的场景很少。应该将 .m2 目录挂载进来，但是 docker build 不像 docker run 一样支持 volume 或者文件系统挂载，所以这里我们重新将依赖下载和编译部分移到了 Dockerfile 之外（依然在 Chronos 调度下的 Docker 容器之中）。考虑到 NFS 网盘读取小文件效率低下，以及版本之间依赖具有延续性，我们将下载到的依赖手动打成 tar 包，在执行 `mvn clean package -U` 之前，检查是否已经有可用的 依赖 tar 包，拉取到本地进行解压，然后更新部分依赖，编译二进制文件。这里因为历史原因，用了 dind(docker in docker)，因此没有将 NFS 网盘上的 .m2 目录挂载到每个 Chronos Task 容器上，因为挂载了也用不上。

第三阶段，我们尝试去除 dind，使用挂载 /var/run/docker.sock 的方式。我们直接将 NFS 上的 .m2 目录挂载到每个 Chronos Task 容器上，但是由于 NFS 小文件读取相当慢，下载依赖耗时比第二阶段 tar 包方式还慢。

第四阶段，这里我们的主想法是将 依赖 deps 放到镜像中，使用 `docker build --cache-from` 的方式（--cache-from 的原理弄清楚后会另开一篇文章进行讲解），但是这里的 cache-from 由于 docker 本身机制的原因，想要使用，只能作为 FROM 的 BASE 镜像（前面已经证明了 依赖层 作为 Dockerfile 的中间层很难被复用，因为 RUN mvn 命令必须要放在 COPY source-code 之后，一旦 source-code 发生变化，那么下面一层带有 下载好的依赖的 docker layer cache 则无法生效），相当于把项目依赖放到了基础镜像中，只是这样做的话，后面的每一层都无法被 layer cache 命中，因为 99.9% 的概率 BASE 镜像虽然名字不变，但是因为依赖更新了导致镜像内容变了，导致后面的 layer cache 均失效。这里本质上和第二阶段一样，只不过将 tar 包放到了 BASE 镜像中，并且由 docker 来管理，之前使用 tar compress 和 decompress 出现文件损坏的概率很大，一旦损坏就需要强制缓存失效重新打包。

改进点，考虑到 BASE 镜像实际上不用发生变化，在运行 `RUN mvn clean package` 的上一步使用 `COPY --from ${cache_img} /root/.m2/repository /root/.m2/repository` 将依赖文件拷贝进来。这样能够使得 `RUN mvn clean package` 之前的 layer cache 几乎都生效。

注意点，maven 基础镜像移除了 /root/.m2 这个 volume，否则的话内容都在 volume 中，不在镜像中，需要手动修改 MAVEN_OPTS 来指定新的 local repository 目录。


## 考虑
1. 是否这样的话，deps 和 compile 事实上又可以放到一起了，说不定比分开更快
2. 使用大镜像作为基础镜像，包含绝大部分依赖，定期更新，节点预热镜像