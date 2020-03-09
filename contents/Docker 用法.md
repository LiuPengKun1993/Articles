# Docker 用法


> Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从 Apache2.0 协议开源。
Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。
容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

下面是 Docker 的一些常见用法：

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

![](https://github.com/liuzhongning/Articles/blob/master/resources/Docker/docker01.jpg)

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
docker rmi 镜像id
```

### Docker 批量删除镜像

Docker 使用一段时间后，可能会存在许多无用的镜像。一个个删除比较麻烦，可以用下面的命令进行批量删除。

```
$ docker rmi $(docker images | grep "none" | awk '{print $3}')

```

`docker images` 会查看所有的镜像；`grep "none"` 命令会筛选所有名字包括 `none` 以及标签为 `none` 的镜像；`awk '{print $3}'` 会处理筛选后的文本，打印所有镜像 `id` 的内容。

![](https://github.com/liuzhongning/Articles/blob/master/resources/Docker/docker02.jpg)

#### 问题汇总

1.报错 `Error response from daemon: conflict: unable to delete 4a67b006c338 (cannot be forced) - image is being used by running container 451b7b600276`

解决方法：使用 `docker rmi -f 镜像id` 进行删除

---

学习资料：https://www.runoob.com/docker/docker-tutorial.html