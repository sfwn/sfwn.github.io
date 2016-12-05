# linux 时区设置

> Date: 2018-03-20
>
> Author: sfwn

# alpine
```text
ENV TZ Asia/Shanghai

RUN echo "http://mirrors.aliyun.com/alpine/v3.6/main/" > /etc/apk/repositories && echo "http://mirrors.aliyun.com/alpine/v3.6/community/" >> /etc/apk/repositories && apk add --no-cache tzdata && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && rm -fr /var/cache/apk/*
```

# debian
```text
ENV TZ "Asia/Shanghai"
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```
