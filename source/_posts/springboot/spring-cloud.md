---
layout: post
title: spring cloud
date: 2018-11-19 13:57:54
tags: 
    - java
    - springcloud
categories: springboot
---

# spring cloud

- 分布式/版本化配置
- 服务注册和发现
- 选路
- 服务对服务呼叫
- 负载平衡
- 断路器
- 分布式消息传递

[cloud文档](http://cloud.spring.io/spring-cloud-static/Camden.SR2/)

## spring cloud config

配置管理中心, 通过 `spring.cloud.config.server.git.uri` 来获取git服务上的配置文档,当然也可配置成本地获取.客户端通过HTTP可以获取对应的 配置资源

```properties
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

### 服务端配置

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
```

### 客户端配置

客户端依赖 `SpringCloud-config-Client`这个jar, 如果你直接引入了 `spring-cloud-starter-config` 则不需要额外添加 `jar`

`bootstrap.properties`

```properties
# 指向服务端的配置中心
spring.cloud.config.uri: http://myconfigserver.com
```