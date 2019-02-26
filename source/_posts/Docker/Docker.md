## centos7 安装 docker

- 一键安装 curl -sSL https://get.docker.com/ | sh
- or https://www.cnblogs.com/wq3435/p/6479768.html

## 启动docker

` systemctl start docker`

## docker常用操作

### 容器

> 查看所有容器

`docker ps -a`



> 查看运行中的容器

`docker ps `



> 停止容器

`docker stop goofy_easley`



> 启动容器

`docker start goofy_easley`



> 进入容器

`docker exec -it 5ecf2637f10b /bin/sh`



> 退出容器

`exit`



### 镜像

> 删除镜像

```
docker images
docker rmi e38bc07ac18e
```

前提是没有容器使用该镜像，如果占用了需要先删除容器，然后删除镜像

```
docker ps -a
docker rm 70cdf9df895e
```

