---
layout: post
title:  "JDBC Update 返回值"
date:   2019-12-06 00:00:00 +0800
tags: [JDBC]
---

# 一、MySQL CLI
```sql
update user set name='test' where id=1;
```

返回：**Rows matched: 1 Changed: 0 Warnings: 0**

# 二、JDBC Client

- executeQuery 方法执行查询SQL语句，返回 ResultSet
- executeUpdate 方法执行更新SQL语句，返回 **rows matched**，不是row changed
- execute 执行SQL语句
  - 如果是查询语句返回true，通过 getResultSet 获取结果
  - 如果是更新语句返回false。通过 getUpdateCount 获取 affected rows（JDBC术语），affected rows实际上是rows matched