---
title: docker serveice
date: 2017-11-12 16:13:57
type: post
tags: 
    - docker
categories: docker
---

# docker serveice

> 扩展了应用程序并启用了负载平衡。

## 分布式应用程序的层次结构

* Stack  堆
* services 服务
* container  

## 什么事 serveice

在分布式应用程序中，应用程序的不同部分称为“服务”。例如，如果您想像一个视频共享站点，它可能包括一个用于在数据库中存储应用程序数据的服务，一个在后台进行视频转码的服务用户上传东西，前端服务等等。
服务只是“生产中的容器”。一个服务只运行一个映像，但它编码映像运行的方式 - 应该使用哪些端口，容器应容器，集装箱该运行多少副本，以便服务具有所需的容量，以及等等。扩展服务会更改运行该软件的容器实例的数量，并在该过程中为服务分配更多的计算资源。

## docker-compose.yml编写

一个docker-compose.yml文件是一个YAML文件，它定义了Docker容器在生产过程中的行为。

```yml
version: "3"
services:
  web:
# 将 60.205.206.164:8001/test:1.2 替换成你的 image ~~REPOSITORY：TAG~~
    image: 60.205.206.164:8001/test:1.2
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "8090:80"
    networks:
      - webnet
networks:
  webnet:
```

## Docker执行以下操作

* 从仓库中拉取 60.205.206.164:8001/test:1.2 这个 image
* 运行该映像的5个实例作为调用的服务web，限制每个实例使用，最多使用10％的CPU（跨所有内核）和50MB RAM
* 将主机上的端口8090映射到web80端口
* 指示web容器通过称为负载平衡网络共享端口8090 webnet。（在内部，集装箱本身将web端口发布到 80端口。）
* webnet使用默认设置（这是一个负载平衡的覆盖网络）来定义网络。

### 运行新的负载均衡应用

* 初始化swarm : `docker swarm init`
* 给应用取一个名字： `docker stack deploy -c docker-compose.yml getstartedlab`
* 通过 `docker service ls` 可以看到 我们的单一服务堆栈在一个主机上运行我们部署映像的5个容器实例。通过 `docker service ps <serviceId>` 可以看到这个服务包含了5个容器，都有独立的 ID
* 页面运行 8090 端口，不断的运行，会发现走了不通的容器 Hostname 显示的ID 一直在变化

### 缩放应用

* 更改docker-compose.yml的replicas值，保存更改并重新运行docker stack deploy命令来缩放应用程
* 通过 `docker service ps <serviceId>` 可以看到这个服务包含容器的个数变化

## Take down the app and the swarm

* `docker stack rm getstartedlab`
> This removes the app, but our one-node swarm is still up and running 
* `docker swarm leave --force`
> Take down the swarm