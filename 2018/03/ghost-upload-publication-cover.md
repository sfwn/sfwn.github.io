# Ghost 无法上传 Publication cover

> Date: 2018-03-20
>
> Author: sfwn

在 Ghost-Settings-General 里上传 Publication cover，发现进度条能走完，但是 ghost-server 没有 api 请求日志。而 Publication icon 和 Publication logo 可以正常上传成功，在 ghost-server 端也能看到 upload api 请求日志。

怀疑是 Publication logo 上传按钮可能没绑定 function，是个假按钮, 在 github 上也确实搜到了相关 issue，但是已经年代久远。

[Effet](https://github.com/Effet) 提醒我可能是 nginx 配置不对。查看了 nginx 日志，发现确实是上传文件的大小超出了最大限制。

nginx 默认上传大小是 1m:
```text
Syntax: client_max_body_size size;
Default: client_max_body_size 1m;
Context: http, server, location

Sets the maximum allowed size of the client request body, specified in the “Content-Length” request header field. If the size in a request exceeds the configured value, the 413 (Request Entity Too Large) error is returned to the client. Please be aware that browsers cannot correctly display this error. Setting size to 0 disables checking of client request body size.
```

而我的 Publication cover 图片大小为 1.01m，nginx `client_max_body_size` 配置修改为 2m 后就 OK 了。