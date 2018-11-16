---
title: 上传 Docker Image
date: 2017-11-20 19:13:57
type: post
tags: 
    - docker
    - 入门
categories: docker
---

# 上传 Docker Image

>再传之前我们需要建立在自己的私有仓库，不然就上传到docker仓库中心了

## 自己server服务器地址

myServer = localhost   下文中`localhost``替换成自己的服务器地址即可

## 建立自己的docker仓库

> 这里我用的是 Nexus3,8001端口是提供给image上传下载用的

```bash
 docker run -d -p 8000:8081 -p 8001:8001 sonatype/nexus3
```

> 默认的仓库会将 资源放在 /var/lib/docker/,随着容器的删除数据也会消失，需要挂载，具体看镜像介绍

```http
https://hub.docker.com/r/sonatype/nexus3/
```

### 登陆 8000 端口后，进入设置，建立自己的 docker仓库。

Repository -> repositories -> Create repository -> docker(hosted)
> name 随便， HTTP或者HTTPS，根具需求填写，我这里勾选HTTP，端口协商8001，与上面暴露出来的端口保持一致，直接save就行了

### 创建一个Dockerfile.用官网的demo

创建一个空目录。将目录（cd）更改到新目录中，创建一个文件 Dockerfile(dockerfile编写可参考官方文档)

```dockerfile
# Use an official Python runtime as a parent image
FROM python:2.7-slim
# Set the working directory to /app
WORKDIR /app
# Copy the current directory contents into the container at /app
ADD . /app
# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt
# Make port 80 available to the world outside this container
EXPOSE 80
# Define environment variable
ENV NAME World
# Run app.py when the container launches
CMD ["python", "app.py"]
```

创建应用体（dockerfile中提及的两个文件）requirements.txt和app.py，并把它们与同一文件夹中Dockerfile。

### 创建requirements.txt

```code
Flask
Redis
```

### 创建app.py

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

## 构建image

```bash
docker build -t friendlyhello .
```

> -t 为你镜像打repository，默认tag为latest(friendlyhello:latest), . 表示存储位置的路径，具体的可以看  docker build --help. 通过 `docker images` 就可以看到刚刚的镜像了

## 登陆自己的仓库

```bash
dokcer login -u ${name} -p ${passwd} ${server地址}
我的是
docker login -u admin -p admin123 ${myServer}:8001
eg: docker login -u admin -p admin123 localhost:8001
```

如果登陆失败，则你需要将你得server地址加入 daemon 文档（即配置镜像加速的位置）， (vi /etc/docker/daemon.json)，比如 docker for windows, 右键右下角docker，选择settting->daemon -> Insercure registries 配置上你服务器的地址加端口即可。（Registry mirrors 就是你配置镜像加速的位置了）

## make tag

给镜像打上tag， docker tag --help 可查看详细用法

```bash
格式 : docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]   docker tag image username/repository:tag
docker tag friendlyhello:latest localhost:8001/test:1.2
docker push localhost:8001/test:1.2
```

至此，登录你的nexus3,在docker仓库就可以看见一个名字为 test,版本为1.2的镜像了

## 检查

执行 `docker search localhost/test:1.2` 看是否存在
删除本地已经存在的镜像 `docker rmi localhost/test:1.2`
执行 `docker run -d -p 8888:80 localhost/test:1.2`, 可观察到下载的状况
浏览器 访问 8888端口