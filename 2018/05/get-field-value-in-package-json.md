# 命令行获取 package.json 中的指定字段值

> Date:  2018-05-18
>
> Author: sfwn

```bash
$ node -p "require('./package.json').scripts.start"
```

这种方式可以解析任何 json。