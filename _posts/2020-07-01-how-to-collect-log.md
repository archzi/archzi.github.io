---
title: 日志收集 
date: 2020-07-01 17:00:00
categories: [笔记]
tags: [总结]     # TAG names should always be lowercase
---



# logstash

- 缺点：
  1. 性能问题 具体 benchmark 可以参考[链接](https://sematext.com/blog/2016/04/25/elasticsearch-ingest-node-vs-logstash-performance/)
  2. 不支持缓存，典型的替代方案还得依赖 kafka 或者Redis 作为中心缓存池

# Flume

Flume基于流式数据的、使用简单的（借助配置文件即可）、健壮的、容错的。

Flume的简单体现在：写一个source、channel、sink之后，一条命令就能操作成功。

Flume、Kafka实时进行数据收集，Storm、Spark实时数据处理，Impala实时查询。

- 实现方式
监控文件 -> 产生事件 -> 发送到channel -> 发送到 sink -> HDFS


- 优点： 
  1. 可恢复性强
  2. 与 kafka 类似，不过不是消息队列，是一种管道流的方式，提供了很多默认的实现，配几个参数就可以用
  3. Flume采集数据到Kafka：自定义Flume的sink组件，将数据从channel中取出，通过kafka的producer写入到Kafka中，可以自定义分区。
  4. 可以直接保存到 HBase
- 缺点
  1. 如果数据要被发送到多个 系统消费的化就必须用 kafka 不能用 flume了


# filebeat
Filebeat 是一个轻量级的日志传输工具，它的存在正弥补了 Logstash 的缺点：Filebeat 作为一个轻量级的日志传输工具可以将日志推送到中心 Logstash。
- 实现方式
- 优点 
  1. 轻量级，不吃资源
- 缺点
  1. 不解析日志，需要 ES 自己解析日志
  2. 日志格式可能需要 存成 json

- 应用场景  1. 日志直接发送到 ES  2. 日志过滤发送到 kafka 或者 redis

# Logagent
Logagent 是 Sematext 提供的传输工具，它用来将日志传输到 Logsene（一个基于 SaaS 平台的 Elasticsearch API），因为 Logsene 会暴露 Elasticsearch API，所以 Logagent 可以很容易将数据推送到 Elasticsearch 。

- 优点
  1. 可以获取 /var/log 下的所有信息，解析各种格式（Elasticsearch，Solr，MongoDB，Apache HTTPD等等），它可以掩盖敏感的数据信息，例如，个人验证信息（PII），出生年月日，信用卡号码，等等。它还可以基于 IP 做 GeoIP 丰富地理位置信息（例如，access logs）。同样，它轻量又快速，可以将其置入任何日志块中。在新的 2.0 版本中，它以第三方 node.js 模块化方式增加了支持对输入输出的处理插件。重要的是 Logagent 有本地缓冲，所以不像 Logstash ，在数据传输目的地不可用时会丢失日志。
  2. 尽管 Logagent 有些比较有意思的功能（例如，接收 Heroku 或 CloudFoundry 日志），但是它并没有 Logstash 灵活。

- 应用场景

  Logagent 作为一个可以做所有事情的传输工具是值得选择的（提取、解析、缓冲和传输）。
# rsyslog
绝大多数 Linux 发布版本默认的 syslog 守护进程，rsyslog 可以做的不仅仅是将日志从 syslog socket 读取并写入 /var/log/messages 。它可以提取文件、解析、缓冲（磁盘和内存）以及将它们传输到多个目的地，包括 Elasticsearch 。可以从此处找到如何处理 Apache 以及系统日志。
- 优点
  1. 性能好  
  2. 各个平台都有 syslog 的 支持，比如 java 的logback 可以执行 syslog 的appender
  3. 解析的规则可以很多，但不影响性能
- 缺点
  1. 配置难

- 应用场景

    rsyslog 适合那些非常轻的应用（应用，小VM，Docker容器）。如果需要在另一个传输工具（例如，Logstash）中进行处理，可以直接通过 TCP 转发 JSON ，或者连接 Kafka/Redis 缓冲。

    rsyslog 还适合我们对性能有着非常严格的要求时，特别是在有多个解析规则时。那么这就值得为之投入更多的时间研究它的配置。

# 初步方案 
1. nginx， docker 等 日志直接 用 filebeat 发送到 ES
2. java 日志 用 syslog 的方式 发送到 logstash