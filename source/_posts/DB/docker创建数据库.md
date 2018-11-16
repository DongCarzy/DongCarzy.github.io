---
title: docker创建数据库
date: 2016-11-14 18:13:57
type: post
tags: 
    - docker
    - mysql
categories: 数据库
---

# docker创建数据库

## mysql

```bash
docker run --name some-mysql -v /home/dxp/mysql/data:/var/lib/mysql  -e MYSQL_ROOT_PASSWORD=root -p 3306:3306 -d mysql:5.7
```