---
title: Dockerfiles的最佳实践
date: 2017-11-16 19:13:57
type: post
tags: 
    - docker
    - 入门
categories: docker
---

# 编写Dockerfiles的最佳做法

Docker可以通过从Dockerfile包含所有命令的文本文件中读取指令来自动构建图像 ，以便构建给定图像所需的顺序。

## 一般准则和建议

### 容器应该是精简的

### 使用.dockerignore文件

在大多数情况下，最好将每个Dockerfile放在一个空目录中。然后，仅添加构建Dockerfile所需的文件。为了增加构建的性能，您可以通过.dockerignore向该目录添加文件来排除文件和目录。此文件支持类似于.gitignore文件的排除模式。

#### 避免安装不必要的包

为了减少复杂性，依赖性，文件大小和构建时间，您应该避免安装额外的或不必要的包

#### 容器单纯化

将应用程序分解成多个容器可以更轻松地水平扩展和重新使用容器。例如，Web应用程序堆栈可能由三个独立的容器组成，每个容器具有自己独特的映像，以解耦的方式管理Web应用程序，数据库和内存中缓存。尽可能地判断容器是否干净，模块化。
如果容器相互依赖，则可以使用Docker容器网络 来确保这些容器可以通信。

#### 最小化层数

你需要找到`dockerfile` 可用性(长期可维护性)之间的平衡,并最大限度地减少其使用的层数。

#### 排序多行参数

只要有可能，通过以字母数字排序多行参数来缓解以后的更改。这将帮助您避免重复的包，并使列表更容易更新。这也使得PR更容易阅读和审查。在反斜杠（`\`）之前添加一个空格。

```dockerfile
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```

#### 构建缓存

在构建映像的过程中，Docker将Dockerfile按照指定的顺序逐步执行每个指令。随着每条指令的检查，Docker将在其缓存中查找可以重用的现有映像，而不是创建一个新的（重复）映像。用`--no-cache=true`该`docker build`命令上,将不会使用缓存
何时会找到匹配的`image`

* 从已经在缓存中的父图像开始，将下一条指令与从该基础图像导出的所有子图像进行比较，以查看其中一个是否使用完全相同的指令构建。如果没有，则缓存无效。
* 在大多数情况下，简单地比较Dockerfile与其中一个子图像的指令是足够的。但是，某些说明需要更多的检查和解释。
* 对于ADD和COPY指令，检查图像中文件的内容，并为每个文件计算校验和。在这些校验和中不考虑文件的最后修改和最后访问的时间。在缓存查找期间，将校验和与现有映像中的校验和进行比较。如果文件（如内容和元数据）中有任何变化，则缓存无效。
* 除了ADD和COPY命令之外，缓存检查不会查看容器中的文件来确定缓存匹配。例如，当处理RUN apt-get -y update命令时，将不会检查在容器中更新的文件以确定是否存在高速缓存命中。在这种情况下，只有命令字符串本身将用于查找匹配。

一旦缓存无效，所有后续Dockerfile命令将生成新的映像，并且高速缓存将不被使用。

### Dockerfile指令

[x] FROM
[x] LABEL
[x] RUN
[x] APT-GET
[x] USING PIPES
[x] CMD
[x] EXPOSE
[x] ENV
[x] ADD or COPY
[x] ENTRYPOINT
[x] VOLUME
[x] USER
[x] WORKDIR
[x] ONBUILD

#### FROM

只要有可能，使用现有的官方存储库作为您的图像的基础。

#### LABEL

您可以为图像添加标签,` (空格)`, `"` 等字符需要转义

```Dockerfile
# Set one or more individual labels
LABEL com.example.version="0.0.1-beta"
LABEL vendor="ACME Incorporated"
LABEL com.example.release-date="2015-02-12"
LABEL com.example.version.is-production=""
# Set multiple labels on one line
LABEL com.example.version="0.0.1-beta" com.example.release-date="2015-02-12"

# Set multiple labels at once, using line-continuation characters to break long lines
LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"
```

#### RUN

为了使您Dockerfile更易于阅读，可理解和可维护，可以将RUN多个行分隔开，用反斜杠分隔的长整型或复杂语句。

#### APT-GET

可能最常见的用例RUN是应用程序`apt-get`。该 `RUN apt-get`命令用来安装软件包
避免，`RUN apt-get upgrade`或者`dist-upgrade`父系image中的许多“必需”程序包将无法在非特权的容器内升级

```dockerfile
RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo
```

apt-get update在RUN语句中单独使用会导致缓存问题和后续apt-get install指令失败,例如:

```dockerfile
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get install -y curl
```

构建图像后，所有层都在Docker缓存中。假设你以后apt-get install通过添加额外的包来修改：

```dockerfile
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get install -y curl nginx
```

Docker将初始化和修改的指令看作是相同的，并重用先前步骤中的缓存。结果apt-get update是不执行，因为构建使用缓存的版本。因为apt-get update没有运行，你的构建可能会有一个过时的版本curl和nginx 包。
使用`RUN apt-get update && apt-get install -y`确保您的`Docker文件`安装最新的软件包版本，无需进一步的编码或手动干预。这种技术被称为`“缓存破坏”`。您还可以通过指定软件包版本来实现缓存清除。这被称为版本固定，版本固定强制构建以检索特定版本，而不管缓存中有什么。这种技术还可以减少由于所需软件包中意外的更改导致的故障。例如：

```dockerfile
RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo=1.3.*
```

下是一个格式正确的RUN指导，显示所有apt-get

```dockerfile
RUN apt-get update && apt-get install -y \
    aufs-tools \
    automake \
    build-essential \
    curl \
    dpkg-sig \
    libcap-dev \
    libsqlite3-dev \
    mercurial \
    reprepro \
    ruby1.9.1 \
    ruby1.9.1-dev \
    s3cmd=1.1.* \
 && rm -rf /var/lib/apt/lists/*
```

#### USING PIPES

一些RUN命令取决于使用管道字符（`|`）将一个命令的输出管道传输到另一个命令的能力

```dockerfile
RUN wget -O - https://some.site | wc -l > /number
```

* Docker使用/bin/sh -c解释器执行这些命令，该解释器仅评估管道中最后一个操作的退出代码以确定成功。在上面的示例中，只要wc -l命令成功，即使wget命令失败，此构建步骤也可以成功并生成新映像。
* 您希望命令由于管道中任何阶段的错误而失败，请先set -o pipefail &&确定一个意外的错误会阻止构建从无意中成功。

```dockerfile
RUN set -o pipefail && wget -O - https://some.site | wc -l > /number
```

> `注意`：并非所有的shell都支持该`-o pipefail`选项。在这种情况下（例如`dash shell`，它是基于`Debian`的映像的默认`shell`），请考虑使用`exec`形式`RUN `来显式选择一个支持该`pipefail`选项的shell 。

```dockerfile
RUN ["/bin/bash", "-c", "set -o pipefail && wget -O - https://some.site | wc -l > /number"]
```

#### CMD

`CMD`指令应用于运行图像包含的软件以及任何参数。
如果图像用于服务，例如Apache和Rails，则可以运行类似的操作` CMD ["apache2","-DFOREGROUND"]`。这种形式的指令是推荐用于任何基于服务的图像。
`CMD`应该给出一个交互式的`shell`，比如`bash`，`python`和`perl`。例如，`CMD ["perl", "-de0"]`，`CMD ["python"]`，或 `CMD [“php”, “-a”]`。使用此表单意味着当您执行类似的操作时`docker run -it python`，您将被放入可用的`shell`中，随时可以使用。

#### EXPOSE

`EXPOSE`指令指示容器将侦听连接的端口.例如，包含Apache Web服务器EXPOSE 80的映像将使用，而包含MongoDB的映像将使用EXPOSE 27017等等

#### ENV

为了使新的软件更容易运行，您可以使用它`ENV`来更新`PATH`容器安装的软件的 环境变量。例如，`ENV PATH /usr/local/nginx/bin:$PATH`将确保`CMD [“nginx”]` 只是工作。
该ENV指令对于提供特定于要集中化的服务的必需环境变量也很有用，例如Postgres's PGDATA。
最后，ENV也可以用来设置常用的版本号，使得版本颠覆更容易维

```dockerfile
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

#### ADD or COPY

`ADD`  和 `COPY` 的功能有点类似,但是一般优先采用`COPY `, 因为它更加透明.
COPY只支持将本地文件基本复制到容器中，同时ADD具有一些不是立即显而易见的功能（如本地仅提取和远程URL支持）。因此，最好的用途ADD是将本地`tar`文件自动提取到图像中，如同 `ADD rootfs.tar.xz /`。
因为图像大小很重要，ADD因此强烈不鼓励使用远程URL提取包; 你应该使用curl或wget替代。这样，您可以删除在解压后不再需要的文件，而不必在图像中添加另一个图层.

##### 错误实例

```dockerfile
ADD http://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```

##### 正确实例

```dockerfile
RUN mkdir -p /usr/src/things \
    && curl -SL http://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```

#### ENTRYPOINT

#### VOLUME

该VOLUME指令应用于公开您的docker容器创建的任何数据库存储区域，配置存储或文件/文件夹。强烈建议您使用图像VOLUME的任何可变和/或用户可维修的部分。

#### USER

#### WORKDIR

为了清晰可靠，您应该永远为您使用绝对路径 WORKDIR。此外，您应该使用，WORKDIR而不是增加的说明，如RUN cd … && do-something难以阅读，排除故障和维护

#### ONBUILD

`ONBUILD`命令在当前`Dockerfile`构建完成后执行。 ONBUILD在导出FROM当前图像的任何子图像中执行。将该ONBUILD命令视为父母Dockerfile给予孩子的指示Dockerfile。
Docker构建`ONBUILD`在子节点`Dockerfile`中的任何命令之前执行命令。
放入ADD或COPY放入时要小心ONBUILD。如果新版本的上下文缺少添加的资源，“onbuild”映像将会严重失败。如上所述，添加单独的标签将有助于通过允许Dockerfile作者做出选择来缓解这一点
