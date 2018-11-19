---
layout: post
title: 问题列表
date: 2018-11-19 14:02:42
tags:
    - java
    - 问题
categories: java
---

# 问题列表

> 平时工作中遇到的问题以及解决办法

* [x] 计算时差问题
* [x] 静态Map，Set用作存储并发
* [x] 获取 `spring.profile.active` 用户

## 计算时差问题

StopWatch 类提供计时器

```java
//获取并启动
StopWatch sw = StopWatch.createStarted();
  = >
StopWatch sw = new StopWatch();
sw.start();

//获取时间
sw.getTime();
or
sw.getTime(TimeUnit.MINUTES);  //TimeUnit表示不同时间形式
```

## 静态Map，Set用作存储并发

采用 `ConcurrentHashMap` 替 `HashMap`
类似 `ConcurrentSkipListSet` 替换 `HashSet`

```java
Map map = new ConcurrentHashMap();
Set<String> set = new ConcurrentSkipListSet<>();
```

* `ConcurrentHashMap`代码中可以看出，它引入了一个“分段锁”的概念，具体可以理解为把一个大的Map拆分成N个小的`HashTable`，根据`key.hashCode()`来决定把key放到哪个HashTable中。
* 在`ConcurrentHashMap`中，就是把Map分成了N个`Segment`，`put`和`get`的时候，都是现根据`key.hashCode()`算出放到哪个`Segment`中。

## 获取 `spring.profile.active` 用户