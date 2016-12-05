# linux 镜像仓库源设置

> Date: 2018-03-20
>
> Author: sfwn

## apline
```text
echo "http://mirrors.aliyun.com/alpine/v3.6/main/" > /etc/apk/repositories && echo "http://mirrors.aliyun.com/alpine/v3.6/community/" >> /etc/apk/repositories
```

## debian 9
```text
sed -i 's/httpredir.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
```

```text
cat > /etc/apt/sources.list <<EOF
deb http://mirrors.aliyun.com/debian/ jessie main non-free contrib
deb http://mirrors.aliyun.com/debian/ jessie-proposed-updates main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ jessie main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ jessie-proposed-updates main non-free contrib
EOF
```

## ubuntu
```text
sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
```

## centos
```text
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```