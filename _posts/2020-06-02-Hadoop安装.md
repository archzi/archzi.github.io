---
title: Hadoop standalone 安装笔记
date: 2020-06-02 17:00:00
categories: [大数据]
tags: [笔记]     # TAG names should always be lowercase
---

## 服务器
```
ip: 192.168.2.106
usr: zl
password: 123456
```
## hadoop 伪集群模式安装

### 版本选择
最新版是 3.3 版本，但是考虑到各个组件的兼容性问题 选择2.10 版本的hadoop

### 安装

1. 根据附录下载 相关文件
2. 解压
   ```bash
    tar -zxvf hadoop-2.10.0.tar.gz
   ```
3. 配置 hadoop 相关命令到系统path 中
   ```
   echo "export HADOOP_HOME=/home/zl/hadoop-2.10.0" >> ~/.bashrc &&
   echo "PATH=\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin:\$PATH" >> ~/.bashrc &&
   source ~/.bashrc
   ```
4. 配置 JAVA_HOME 环境变量
   命令 略

5. 测试一下命令，验证安装是否正常
   ```
   hadoop
   ```
### 配置 hdfs  伪集群模式
1. ${HADOOP_HOME}/etc/hadoop/core-site.xml 中添加:
    ```xml
    <configuration>
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://localhost:9000</value>
      </property>
    </configuration>
    ```
2. ${HADOOP_HOME}/etc/hadoop/hdfs-site.xml:
    ```xml
    <configuration>
    <property>
              <name>dfs.replication</name>
              <value>1</value>
          </property>
      <property>
            <name> dfs.namenode.name.dir</name>
          <value>/home/zl/hadoop-data/namenode</value>
    </property>
    <property>
    <name>dfs.datanode.data.dir</name>
    <value>/home/zl/hadoop-data/datanode</value>
    <property>
    </configuration>
    ```

3. 配置 ssh
    ```bash
    ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa &&
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys  &&
    chmod 0600 ~/.ssh/authorized_keys
    ```
### 启动hdfs
1. 执行 格式化 hdfs
    ```
    bin/hdfs namenode -format
    ```
2. 启动
   ```
   sbin/start-dfs.sh
   ```
3. 关闭
   ```
   sbin/stop-dfs.sh
   ```

root 用户启动需要设置 环境变量
```bash
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```
### 配置 yarn 伪集群模式

1. ${HADOOP_HOME}/etc/hadoop/mapred-site.xml 中添加
   ```xml
   <configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    </configuration>
   ```
2. ${HADOOP_HOME}/etc/hadoop/yarn-site.xml 中添加
   ```xml
    <configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    </configuration>
   ```
3. 启动
   ```bash
   start-yarn.sh
   ```
4. 关闭
   ```
   stop-yarn.sh
   ```
> web 页面： http://localhost:8088/

### web 端口

|组件|配置|默认值|
|---|---|---|
|NameNode|http://nn_host:port/|Default HTTP port is 50070.
|ResourceManager|	http://rm_host:port/|Default HTTP port is 8088.
|MapReduce | JobHistory Server |	http://jhs_host:port/	Default HTTP port is 19888.

## Hbase 安装 (Pseudo-distributed)
### 版本选择
最新版 2.3.0

### 安装
1. 解压
   ```
   tar -zxvf hbase-2.3.0-bin.tar.gz
   ```
2. 配置 java_home 略
3. 配置 PATH
   ```
   echo "export HBASE_HOME=/home/zl/hbase-2.3.0" >> ~/.bashrc &&
   echo "PATH=\$HBASE_HOME/bin:\$PATH" >> ~/.bashrc &&
   source ~/.bashrc
   ```

### 配置 hbase 伪集群

1. conf/hbase-site.xml 中添加
    ```xml
      <configuration>
      <property>
      <name>hbase.cluster.distributed</name>
      <value>true</value>
      </property>
      <property>
      <name>hbase.rootdir</name>
      <value>hdfs://localhost:9000/hbase</value>
      </property>
      </configuration>
    ```
2. 启动 hbase bin/start-hbase.sh

3. web 页面访问 http://localhost:16010


## sqoop 导入相关依赖
用 sqoop 导入 包 版本 依赖

1.  hadoop 2.x
2.  hbase 2.x
3.  hbase-client 1.12.x
4.  metrics-core 2.2.0

jar 包 放到 sqoop lib 中

参考 链接
1. https://www.cnblogs.com/jdbc-mydql/p/8489961.html

## 附：相关链接

1. [hadoop-doc](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)

2. [hadoop 2.10 安装包](https://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.10.0/hadoop-2.10.0.tar.gz)

3. [Hbase 2.2.5 安装包](https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/2.2.5/hbase-2.2.5-bin.tar.gz)



