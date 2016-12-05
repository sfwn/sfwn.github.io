# docker container cp when destination already exist

> Date:  2018-04-17
>
> Author: sfwn

```bash
mkdir -p ${BUILDING_ROOT}/app/public

docker container cp ${temp_container}:/app/public ${BUILDING_ROOT}/app/public
docker container cp ${temp_container}:/app/nginx.conf.template ${BUILDING_ROOT}/app
```

如果 app/public 未创建，则 cp 之后正确

如果 app/public 提前创建，则 cp 之后目录结构为 app/public/public/...