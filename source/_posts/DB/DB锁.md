---
title: 锁表与解锁
date: 2017-06-14 16:13:57
type: post
tags: 锁表与解锁
categories: 数据库
---

# 锁表与解锁

## ORACLE

### 查询锁表情况

```sql
SELECT object_name, machine, s.sid, s.serial#
FROM gv$locked_object l, dba_objects o, gv$session s
WHERE l.object_id = o.object_id
AND l.session_id = s.sid
```

### 锁表

```sql
LOCK TABLE CAM_ACCOUNT_MST IN SHARE MODE ;
```

### 解锁

```sql
alter system kill session 'sid, serial#'
ALTER system kill session '17, 23019'
```

### 查询锁表情况2

```SQL
SELECT l.session_id sid,
  s.serial#,
       l.locked_mode 锁模式,
       l.oracle_username 登录用户,
       l.os_user_name 登录机器用户名,
       s.machine 机器名,
       s.terminal 终端用户名,
       o.object_name 被锁对象名,
       s.logon_time 登录数据库时间
FROM v$locked_object l, all_objects o, v$session s
WHERE l.object_id = o.object_id
      AND l.session_id = s.sid
ORDER BY sid, s.serial#;
```