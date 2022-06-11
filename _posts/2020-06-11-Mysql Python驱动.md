---
title: MySQL python 驱动 选择
date: 2020-06-11 17:00:00
categories: [Python]
tags: [笔记]     # TAG names should always be lowercase
---

因为 python 总所周知的历史原因，有很多包可以选择，这边一句话总结一下，希望有帮助

## python2
### MySQL-Python
mysql-python 是 python2 时代的连接mysql 的package，底层用C 实现，用python 包装成 python package,安装的时候需要编译，也可以下载二进制文件进行安装，
但是需要注意的是： mysql-python 只 支持 myqsl 3.23- 5.5 的版本和python 2.4-2.7版本，如果有历史项目无法迁移的，时候需要注意。

### mysql-connector-python
mysql-connector-python 是oracle 收购 mysql 之后为了解决python mysql adapter 混乱的问题开发的一个package，用纯python 实现，性能没有 mysql-python 好，但是优点是 支持很广泛，从python2 到3 都可以，也支持连接池，线程安全，属于可靠的类别

## python3

### mysqlclient
mysqlclient  是 mysql-python 的一个 mysqldb1分支的的clone 版本，在 msyqldb1 的基础上 开发了支持 python3 的版本，并且兼容之前的mysql-python。
不同的python 版本 和 mysql 版本可以下载不同的版本支持。

### pymysql
跟 mysql-connector-python 一样是个 纯python 实现的adapter，但是存在线程安全的问题，并且不支持连接池。属于历史的眼泪了，基本上已经不更新了。

## 参考
- [django 官方](https://docs.djangoproject.com/en/3.0/ref/databases/#mysql-db-api-drivers)
- [知乎](https://www.zhihu.com/question/268449231)
- [跑分](https://github.com/methane/mysql-driver-benchmarks)
- [stackoverflow](https://stackoverflow.com/questions/43102442/whats-the-difference-between-mysqldb-mysqlclient-and-mysql-connector-python)
