---
title: Docker安装
date: 2017-11-10 16:13:57
type: post
tags: 
    - docker
categories: docker
---

# Docker安装

## Ubuntu 14.04 16.04 (使用apt-get进行安装)

- Step 1: 安装必要的一些系统工具

```shell
  sudo apt-get update
  sudo apt-get -y install apt-transport-https ca-certificates
  curl software-properties-common
```

- Step 2: 安装GPG证书

```shell
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

- Step 3: 写入软件源信息

```shell
 sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```

- Step 4: 更新并安装Docker-CE

```shell
  sudo apt-get -y update
  sudo apt-get -y install docker-ce 
```

### 安装指定版本的Docker-CE:

- Step 1: 查找Docker-CE的版本:

```shell
apt-cache madison docker-ce # docker-ce | 17.03.1~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
docker-ce | 17.03.0~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
```

- Step 2: 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.1~ce-0~ubuntu-xenial)

```shell
sudo apt-get -y install docker-ce=[VERSION]
```

- Step 3: 安装最新版本的Docker-CE:

```shell
 sudo apt-get -y install docker-ce
```

## CentOS 7 (使用yum进行安装)

- step 1: 安装必要的一些系统工具

```shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

- Step 2: 添加软件源信息

```shell
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

- Step 3: 更新并安装 Docker-CE

```shell
sudo yum makecache fast
sudo yum -y install docker-ce
```

- Step 4: 开启Docker服务

```shell
  sudo service docker start
```

### 安装指定版本的Docker-CE

- Step 1: 查找Docker-CE的版本

```shell
 yum list docker-ce.x86_64 --showduplicates | sort -r
 Loading mirror speeds from cached hostfile
 Loaded plugins: branch, fastestmirror, langpacks
 docker-ce.x86_64 17.03.1.ce-1.el7.centos docker-ce-stable
 docker-ce.x86_64 17.03.1.ce-1.el7.centos @docker-ce-stable
 docker-ce.x86_64 17.03.0.ce-1.el7.centos docker-ce-stable
 Available Packages
```

- Step2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)

```shell
sudo yum -y install docker-ce-[VERSION]
```

- 安装校验 查看docker信息

```shell
docker version
```

- 运行一个简单的  docker image。  会输出一个 hello world!

```shell
docker run 'hello-world'
```