---
title: Docker学习备忘
categories:
- 服务端
- 运维工具
- Docker
tags:
- Docker
abbrlink: qwlXi3BTkHM8Jodq9bA2Ig
date: 2016-06-22 16:47:21
---

![docker](http://7xk7wj.com1.z0.glb.clouddn.com/blog_docker.png)


## docker 需要注意的事情：

> Docker Quickstart Terminal可以进行docker命令的操作，但使用操作系统的命令行工具进行docker命令的操作会报错，docker-machine命令却可以正常使用，这是因为在环境变量中缺少docker的指向。

<!-- more -->

1. 首先，保证docker-machine虚拟机处于运行状态
2. 在操作系统自带命令行工具环境下 `docker-machine env`
3. 复制最后一行rem的命令,例如windos系统下是：` @FOR /f "tokens=*" %i IN ('docker-machine env') DO @%i`
4. 貌似每次重启系统都需要操作一次？

> 如何理解容器和镜像。

1. 镜像（Image），通过Docker Hub下载或`docker commit`创建的只读层

2. 容器（Container），通过`run`，在镜像的基础上创建一个改动层。容器可以运行、停止、删除、...，已经停止的容器中的数据并不会消失，而是被存储在相应的“改动层”中。

  *总结：容器= 镜像+改动层*

> docker 搭建开发环境实战

... ...  留位填坑

## docker的一些命令：

* 查看镜像列表：`docker images`

* 查看运行中容器列表： `docker ps`

* 查看所有容器列表：`docker ps -a`

* 运行一个容器：`docker run -it ubuntu /bin/bash`

* 指定容器的名称： 在 `docker run` 的时候用 `--name` 参数指定容器的名称。

* 退出容器：`exit`

* 删除容器：`docker rm ***` ， 删除镜像：`docker rmi ***`

* 镜像改动历史： `docker history ***`

本文参考的文章：
1. [深入 Docker：容器和镜像](https://segmentfault.com/a/1190000002766882)

2. [理解docker命令](http://dockone.io/article/783)

3. [利用Docker构建开发环境](http://tech.uc.cn/?p=2726)
