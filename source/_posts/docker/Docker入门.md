---
title: Docker入门
date: 2017-11-11 19:13:57
type: post
tags: 
    - docker
    - 入门
categories: docker
---

# Docker入门

## 搜索docker镜像

```dokcer
docker search tutorial
```

## 下载容器

> 可能可能需要配置镜像加速

```docker
docker pull learn/tutorial
```

docker容器可以理解为在沙盒中运行的进程。这个沙盒包含了该进程运行所必须的资源，包括文件系统、系统类库、shell 环境等等。但这个沙盒默认是不会运行任何程序的。你需要在沙盒中运行一个进程来启动某一个容器。这个进程是该容器的唯一进程，所以当该进程结束的时候，容器也会完全的停止。

## 在容器中安装新的程序

tutorial镜像是基于ubuntu的，所以你可以使用ubuntu的apt-get命令来安装ping程序：apt-get install -y ping。

```shell
docker run learn/tutorial apt-get install -y ping
```

## 保存对容器的修改

当你对某一个容器做了修改之后（通过在容器中运行某一个命令），可以把对容器的修改保存下来，这样下次可以从保存后的最新状态运行该容器。docker中保存状态的过程称之为committing，它保存的新旧状态之间的区别，从而产生一个新的版本

1. 运行docker commit，可以查看该命令的参数列表。
2. 你需要指定要提交保存容器的ID。(译者按：通过docker ps -l 命令获得)
3. 无需拷贝完整的id，通常来讲最开始的三至四个字母即可区分。（译者按：非常类似git里面的版本号)

```bash
docker commit 698 learn/ping
```

## 运行新的镜像

在新的镜像中运行 `ping www.baidu.com` 命令

```docker
docker run lean/ping ping www.baidu.com
```

## 发布docker镜像

我们可以将其发布到官方的索引网站。还记得我们最开始下载的learn/tutorial镜像吧，我们也可以把我们自己编译的镜像发布到索引页面，一方面可以自己重用，另一方面也可以分享给其他人使用

# 配置镜像加速

* 国内做Docker镜像站的还蛮多的，阿里、163、DaoCloud这些都是比较好的镜像站地址

## 比如

### win10 上 Docker for Windows

1. 阿里的镜像站地址为：`https://dev.aliyun.com/search.html` ，访问该地址然后登陆阿里云账号—->在产品控制台—>Docker镜像仓库 –>镜像库—>Docker Hub 镜像站点 Copy “您的专属加速器地址”
2. 右键电脑右下角的Docker 图标–>Settings–>Daemon—> 将加速器地址复制到该页面上的文本框中，点击Apply 然后等待Docker重启，重启完毕就可以使用新的Docker镜像源了
3. 其余几个机器的配置方式在参考  阿里云  `https://dev.aliyun.com/search.html`