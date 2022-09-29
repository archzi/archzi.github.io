---
title: CDH 平台安装笔记
date: 2020-06-01 17:00:00
categories: [大数据]
tags: [笔记]     # TAG names should always be lowercase
---


## 集群准备

物理机配置
cpu: core:10 * 4
内存： 16G
硬盘： 1T

### 安装 vmware esxi 虚拟机平台

[参考](https://blog.csdn.net/z136370204/article/details/89196909)


### esxi 安装虚拟机
所有选择 centos7 最小安装版本

|主机名|cpu|内存|硬盘|
|---|---|---|---
|master|8|6G|300G|
|slave-1|8|4G|100G|
|slave-2|8|4G|100G|

### 安装 CDH

#### 安装前准备
- 更改hostname
  1. 更改host name
     106、107、108 上都添加如下host
     ```
      192.168.2.106  master.jizhang.com  master
      192.168.2.107  slave-1.jizhang.com slave-1
      192.168.2.108  slave-2.jizhang.com slave-2
     ```

     106 上执行  `hostnamectl set-hostname master.jizhang.com`
     107 上执行 `hostnamectl set-hostname slave-1.jizhang.com`
     108 上执行 `hostnamectl set-hostname slave-2.jizhang.com`
  2. 分别 编辑 `/etc/sysconfig/network` 为以下内容
     ```bash
     HOSTNAME=master/slave-1/slave-2.jizhang.com # 只需要写 master 或者slave
     ```

  3. 验证hostname
     - 执行 host -v -t A $(hostname) （报错安装 `yum install bind-utils`）,查看是否为 对应的hostname
     - uname -a 中是否有 对应的hostname
- 关闭防火墙

   centos 执行 `sudo systemctl disable firewalld && sudo systemctl stop firewalld`
- 设置 SELinux 模式
  1. 执行命令 `getenforce` 如果输出是 `Enforcing`,继续，否则跳过
  2. 打开文件 `/etc/selinux/config`中设置 `SELINUX=permissive`，保存退出
  3. 重启服务器，并执行命令`setenforce 0`
  4. CDH 安装完成可以重新把 `SELINUX=permissive`g改回`SELINUX=Enforcing`并执行命令`setenforce 1`重新激活`Enforcing`模式
- 设置 NTP SERVER

  centos 默认安装好了就配置好了，需要配置 参考[官方文档](https://docs.cloudera.com/documentation/enterprise/6/latest/topics/install_cdh_enable_ntp.html)

#### 安装 CM
- 下载 CM repo 文件
  1. 执行
    ```bash
    sudo wget https://archive.cloudera.com/cm6/6.3.1/redhat7/yum/cloudera-manager.repo -P /etc/yum.repos.d/
    ```
  2. import repo
    下面的 username 和 password 在 cdh 官网注册一个
    ```
    sudo rpm --import https://username:password@archive.cloudera.com/p/cm6/6.3.3/redhat7/yum/RPM-GPG-KEY-cloudera
    ```

### CM 数据库迁移
CM 自带 PostgreSQL 数据库，需要从PostgreSQL 数据库迁移到 mysql 数据，


### 常用命令
```bash
sudo service cloudera-scm-agent stop #停止 agent
sudo service cloudera-scm-server stop  #停止 web 页面
service cloudera-scm-server-db stop   #停止 内置 db
sudo systemctl stop supervisord
```



