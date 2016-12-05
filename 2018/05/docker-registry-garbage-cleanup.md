# docker registry garbage cleanup

> Date:  2018-05-26
>
> Author: sfwn

脚本在 gist 上：
https://gist.github.com/sfwn/7453e78be0374b3d53f1e44f5bb8beef

注意：

- 需要运行 registry garbage-collection 命令真正删除。
https://docs.docker.com/registry/garbage-collection/

    > You should ensure that the registry is in read-only mode or not running at all. If you were to upload an image while garbage collection is running, there is the risk that the image’s layers are mistakenly deleted leading to a corrupted image.
    > 
    > This type of garbage collection is known as stop-the-world garbage collection.