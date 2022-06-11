---
title: HUE 简单部署
date: 2020-06-03 17:00:00
categories: [大数据]
tags: [笔记]     # TAG names should always be lowercase
---

## 部署

-  docker 镜像拉取
```
docker pull gethue/hue:latest
```

- 启动命令
```
docker run -d -p 8888:8888 -v /usr/share/hue/desktop/conf:/usr/share/hue/desktop/conf  --name hue --restart always gethue/hue:latest
```
>说明： 挂载 desktop/conf 中的 hue.ini 文件到 宿主机目录下，可以很方便的修改配置文件并 重启： docker container restart hue


## 配置hue 数据库：

修改 /desktop/conf/hue.ini 配置文件中 database 配置为如下配置：
```bash
engine=mysql
host=192.168.2.99
port=3306
user=root
password=BOOT-xwork1024
name=hue
```

## 添加数据源

### Mysql
1.  /desktop/conf/hue.ini 配置文件中 librdbms.databases 模块中添加如下配置：
   ```
    [[[mysql]]]
      # Name to show in the UI.
      nice_name="99MySQL"
      # name=spider
      engine=mysql
      host=192.168.2.99
      port=3306
      user=root
      password=BOOT-xwork1024
      options={"charset":"utf8mb4"} # 解决中文乱码
   ```
2.  notebook.interpreters 中添加如下配置
   ```
    [[[mysql]]]
       name = 99MySQL
       interface=rdbms
   ```

### Hbase

待添加

### hive

待添加
## 权限设置

hue 的权限用的是 django 框架的权限模块，也web 页面上可以设置 组权限，并将响应的用户添加到组中。



