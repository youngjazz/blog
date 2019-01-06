# centos7 安装 docker
- 一键安装 curl -sSL https://get.docker.com/ | sh
- or https://www.cnblogs.com/wq3435/p/6479768.html

# systemctl start docker

# docker 安装 nginx
```bash
查看nginx镜像列表
$ docker search nginx

拉去最新nginx镜像
$ docker pull nginx
```

# docker 运行 nginx

```
docker run --detach \
    --name staticserver \
    --publish 80:80 \
    -v /data/websites/blog:/usr/share/nginx/html:rw \
    -v /data/config:/etc/nginx:rw \
    -d nginx
```
```
docker run --detach \
    --name staticserver \
    --publish 80:80 \
    -v /data/websites/blog:/usr/share/nginx/html:rw \
    -v /data/config/nginx.conf:/etc/nginx/nginx.conf:rw \
    -v /data/config/servers:/etc/nginx/servers:rw \
    -v /data/logs/error.log:/var/log/nginx/error.log:rw \
    -d nginx
```

## 如果被占用

docker ps -a 查看容器id
docker rm id


## 进入一个运行中的容器
docker exec -it 775c7c9ee1e1 /bin/bash

## 异常
`
Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.
`