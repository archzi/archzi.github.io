---
title: Flink app 部署
date: 2020-06-12 17:00:00
categories: [大数据, Flink]
tags: [笔记]
---

### 部署前检查工作
1. 显式设置最大并行度
2. 给所有的 operator 设置 uuid (如果 对 state 敏感的话)
3. 设置 正确的状态储存后端，避免用 MemoryStateBackend  FsStateBackend or RocksDBStateBackend
4. 配置 JobManager 高可用     Standalone Cluster HA  or   YARN Cluster HA

## standalone
### 环境配置

#### ssh
两台机器
- master.jizhang.com
- dev-1.jizhang.com
互相配置 ssh 免密登录并添加 hosts


#### java
以下两种方式二选一
- 配置 `JAVA_HOME`
- `conf/flink-conf.yaml` 中 配置 env.java.home


#### flink 设置
##### 必须
- conf/flink-conf.yml 中
```
jobmanager.rpc.address: master  # 设置为 master 节点 host
```
- conf/masters 中
```
master:8081
```
- conf/workers 中
```
master
dev-1
```
##### 可选
```
jobmanager.rpc.port: 6123
jobmanager.heap.size: 1024m   # 设置jvm heap 大小
taskmanager.memory.process.size: 1568m
taskmanager.numberOfTaskSlots: 3   # 设置 taskmanager 中 的 task slot 影响并行 task 数量 一般为 cpu 的数量，可根据 task 的具体情况而定。
parallelism.default: 3   # 同上
```

相同配置文件 从 master 节点 copy 到 其他节点

### 启动和停止
```
bin/start-cluster.sh # 启动
bin/stop-cluster.sh   # 停止
```

### Job manager 高可用
待添加

###
