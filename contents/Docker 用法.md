# Docker 用法

### 拉取镜像

```
sudo docker pull NAME[:TAG]
sudo docker pull centos:latest
```

### 查看镜像列表，列出本地的所有 images

```
sudo docker images [OPTIONS] [NAME]
sudo docker images
```

[](https://github.com/liuzhongning/Articles/blob/master/resources/Docker/docker01.jpg)

```
REPOSITORY：表示镜像的仓库源
TAG：镜像的标签
IMAGE ID：镜像ID
CREATED：镜像创建时间
SIZE：镜像大小
```

### 查看容器列表，可看到我们创建过的所有 container

```
sudo docker ps [OPTIONS]
sudo docker ps -a
```

### 启动 Container

```
sudo docker run 镜像id
sudo docker run -it 镜像id /bin/bash
```

### 停止 Container

```
sudo docker stop 镜像id
```

### 重启 Container

```
sudo docker restart 镜像id
```

### 通过 ID 删除镜像

```
docker rmi ee7cbd482336
```

### Docker 批量删除镜像

Docker 使用一段时间后，可能会存在许多无用的镜像。一个个删除比较麻烦，可以用下面的命令进行批量删除。

```
$ docker rmi $(docker images | grep "none" | awk '{print $3}')

```

`docker images` 会查看所有的镜像；`grep "none"` 命令会筛选所有名字包括 `none` 以及标签为 `none` 的镜像；`awk '{print $3}'` 会处理筛选后的文本，打印所有镜像 `id` 的内容。

[](https://github.com/liuzhongning/Articles/blob/master/resources/Docker/docker02.jpg)
