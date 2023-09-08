---
title: Docker使用记录
date: 2023-09-08 11:53:02
description: docker使用过程中的记录
tags:
- docker
categories:
- docker
mathjax: true
---

# Docker 使用

## 可视化工具管理

可视化工具Portainer，可以方便地管理Docker

```bash
sudo systemctl restart docker
sudo docker pull portainer/portainer
sudo docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock --restart=always --name prtainer portainer/portainer

```

### 使用方法 （待补充）

## Dockerfile

### COPY指令

- 作用：复制内容到镜像
- 格式：  COPY <src> <dest> 

- 详解：复制本地主机的 <src>下内容到镜像中的 <dest>，目标路径不存在时，会自动创建。
- <src>：可以是 Dockerfile 所在目录的一个相对路径（文件或目录）
- <dest>：可以是镜像内绝对路径，或者相对于工作目录（WORKDIR）的相对路径
- 路径：支持正则表达式，  

### WORKDIR指令

- 切换到镜像中的指定路径，设置工作目录
- 在 WORKDIR 中需要使用绝对路径，如果镜像中对应的路径不存在，会自动创建此目录
- 一般用 WORKDIR 来替代  切换目录进行操作的指令

RUN cd <path> && <do something>

- WORKDIR 指令为 Dockerfile 中跟随它的任何 RUN、CMD、ENTRYPOINT、COPY、ADD 指令设置工作目录
- 如果 WORKDIR 不存在，即使它没有在任何后续 Dockerfile 指令中使用，它也会被创建

### RUN指令

- 作用：在Shell中运行命令，Linux默认的shell是/bin/sh -c 
- 两种形式如下

```dockerfile
RUN <command>
RUN ["executable", "param1", "param2"] 
```

## 开放ssh端口

### 端口映射

使用**-P**参数时，Docker会随机映射一个端口到内部容器开放的网络端口

使用**-p**参数时，Docker可以指定映射端口，具体支持的格式有：

```
IP:HostPort:ContainerPort
IP:ContainerPort
HostPort:ContainerPort
```

### 查看端口映射

```dockerfi
docker port CONTAINER [PRIVATE_PORT[/PROTO]]
```

## 开放端口

#### 方法一：删除原有容器，重新建新容器

这个解决方案最为简单，把原来的容器删掉，重新建一个。当然这次不要忘记加上端口映射。优点是简单快捷，在测试环境使用较多。缺点是如果是[数据库](https://cloud.tencent.com/solution/database?from_column=20065&from=20065)镜像，那重新建一个又要重新配置一次，就比较麻烦了

#### 方法二：利用docker commit新构镜像

docker commit：把一个容器的文件改动和配置信息commit到一个新的镜像。这个在测试的时候会非常有用，把容器所有的文件改动和配置信息导入成一个新的docker镜像，然后用这个新的镜像重起一个容器，这对之前的容器不会有任何影响。

1、停止docker容器

```javascript
docker stop container01复制
```

2、commit该docker容器

```javascript
docker commit container01 new_image:tag复制
```

3、用前一步新生成的镜像重新起一个容器

```javascript
docker run --name container02 -p 80:80 new_image:tag复制
```

这种方式的优点是不会影响统一[宿主机](https://cloud.tencent.com/product/cdh?from_column=20065&from=20065)上的其他容器，缺点是管理起来显得比较乱。

#### 方法三：修改文件端口，重启docker服务

**1.停止docker(`一定`要先停止dokcer，不然直接修改配置文件`不会生效`)**

```javascript
systemctl stop docker
```

  **2.修改这个容器的hostconfig.json文件中的端口（如果config.v2.json里面也记录了端口，也要修改）**

 配置文件路径`/var/lib/docker/containers/`

```
 如果之前没有端口映射, 应该有这样的一段: “PortBindings”:{} 增加一个映射, 这样写: “PortBindings”:{“8080/tcp”:[{“HostIp”:””,“HostPort”:“60000”}]} 前一个数字是容器端口, 后一个是宿主机端口。将宿主机的60000端口映射到容器的8080端口 而修改现有端口映射更简单, 把端口号改掉就行。
```

**hostconfig.json文件**

![在这里插入图片描述](https://ask.qcloudimg.com/http-save/yehe-8223537/sjq500tiav.png)

**config2.json**

![在这里插入图片描述](https://ask.qcloudimg.com/http-save/yehe-8223537/93odymtuex.png)

**3.重启docker服务**

```
systemctl restart docker
```

**4.查看配置项是否完成**

```
docker inspect  CONTAINER ID
```

## 可视化

```dockerfile
FROM osrf/ros:melodic-desktop-full
# nvidia-container-runtime
ENV NVIDIA_VISIBLE_DEVICES \
${NVIDIA_VISIBLE_DEVICES:-all}

ENV NVIDIA_DRIVER_CAPABILITIES \
${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics

```

```bash
sudo xhost +local:
sudo docker run -it --device=/dev/dri --group-add video --volume=/tmp/.X11-unix:/tmp/.X11-unix  --env="DISPLAY=$DISPLAY"  --name=rocker osrf/ros:melodic-desktop-full  /bin/bash

```

## 问题

**缺少NVIDIA GPG Key**

**报错**

```
7.094 Reading package lists...
7.640 W: GPG error: https://developer.download.nvidia.cn/compute/cuda/repos/ubuntu1804/x86_64  InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY A4B469963BF863CC
7.640 E: The repository 'https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64  InRelease' is no longer signed.

```

**解决办法**

```dockerfile
RUN rm /etc/apt/sources.list.d/cuda.list
RUN rm /etc/apt/sources.list.d/nvidia-ml.list
RUN apt-key del 7fa2af80
RUN apt-get update && apt-get install -y --no-install-recommends wget
RUN wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-keyring_1.0-1_all.deb
RUN dpkg -i cuda-keyring_1.0-1_all.deb

```

