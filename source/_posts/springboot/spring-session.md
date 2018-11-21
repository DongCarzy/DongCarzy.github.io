---
layout: post
title: spring-session
date: 2018-11-19 13:57:54
tags: 
    - java
    - spring
categories: springboot
---

# spring-session

Spring Session提供了用于管理用户会话信息的API和实现。

## 特征

`spring session` 使得支持集群会话变得很容易,而不依赖任何特定的应用程序容器,并且还提供以下信息

* `HttpSession` - 允许应用程序容器(Tomcat) 中的 `HttpSession`, 在`RESTFUL`模式中`SpringSession`允许在标头中提供`SessionId`
* `WebSocket` - 提供在接收`WebSocket`消息时保持`HttpSession`活动的能力
* `WebSession` - 允许以与应用程序容器无关的方式替换`SpringWebFlux`的`WebSession`

## 结构

* `springSession core` -  提供`springSession`功能的核心API
* `springSession Data Redis` -  提供由`Redis`和配置支持的`SessionRepository`和`ReactiveSessionRepository` - 实现
* `SpringSession JDBC` -  提供由`关系数据库`和配置支持的 `SessionRepository`实现
* `Spring Session Hazelcast` -   提供由`Hazelcast`和配置支持的 `SessionRepository`实现

### Spring Session

`Spring Session` 中透明的继承了`HttpSession`, 我们可以简单粗暴的使用 `SpringSession`替换我们传统的`HttpSession`.

### SpringSession优势

* 方便做集群会话,儿不限制使用特定的应用容器
* `SpringSession` 支持在单个浏览器实例中管理多个用户会话
* 在`RESTFUL`模式中`SpringSession`允许在标头中提供`SessionId`

### SpringSession继承

* redis
* Pivotal Gemfire
* JDBC
* Mongo
* Hazelcast

#### HttpSession with Redis

这是一个简单的案例,具体配置可以参考官网, 一下是一个spring的配置类,主要做了两件事

```java
@EnableRedisHttpSession 
public class Config {

        @Bean
        public LettuceConnectionFactory connectionFactory() {
                return new LettuceConnectionFactory(); 
        }
}
```

1. `@EnableRedisHttpSession` 创建一个名为`springSessionRepositoryFilter`的过滤器,负责替换`HttpSession`变为`SpringSession`的实现,并且启动`Redis`对 `session`管理的支持
2. 创建`RedisConnectionFactory`,它负责将`SpringSession`连接到 `Redis Server`, 此时的配置采用的默认配置, 即`localhost:6379`. 
> `spring data redis` 默认支持 `Lettuce` and `Jedis`等 `Redis`连接工具

### HttpSession 是如何工作得

#### SessionRepositoryRequestWrapper

我们知道,我们在操作的`HttpSession`和`HttpServletRequest`是两个接口,也正是因为这样,出现了 `SessionRepositoryRequestWrapper`对象,它继承于`HttpServletRequestWrapper`, 并重写了 `getSession`方法

```java
public class SessionRepositoryRequestWrapper extends HttpServletRequestWrapper {

        public SessionRepositoryRequestWrapper(HttpServletRequest original) {
                super(original);
        }

        public HttpSession getSession() {
                return getSession(true);
        }

        public HttpSession getSession(boolean createNew) {
                // create an HttpSession implementation from Spring Session
        }

        // ... other methods delegate to the original HttpServletRequest ...
}
```

返回`HttpSession`都被覆盖了。所有其他方法都由`HttpServletRequestWrapper`并简单地将其委托给原来的`HttpServletRequest`执行。

#### SessionRepositoryFilter

下面是部分的伪代码

```java
public class SessionRepositoryFilter implements Filter {

        public doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
                HttpServletRequest httpRequest = (HttpServletRequest) request;
                SessionRepositoryRequestWrapper customRequest =
                        new SessionRepositoryRequestWrapper(httpRequest);

                chain.doFilter(customRequest, response, chain);
        }

        // ...
}
```

通过`SessionRepositoryFilter`过滤器,在这里去替换 `HttpServletRequest` 为 `SessionRepositoryRequestWrapper`,注意的是, `Spring Session`的过滤器`SessionRepositoryFilter`必须放在与`HttpSession` 交互的任何东西前面.