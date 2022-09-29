---
title: Optimizing Kafka consumers (kafka consumers(消费者)调优)(翻译)
date: 2020-06-01 17:00:00
categories: [中间件]
tags: [[翻译学习],[挖坑]]     # TAG names should always be lowercase
---
[原文](https://strimzi.io/blog/2021/01/07/consumer-tuning/)

## 引言
上篇[《微调 Kafka 生产者》](https://strimzi.io/blog/2020/10/15/producer-tuning/)文章中，我们给出了几点可以改进 Kafka 生产端的建议。这篇文章我们将要来检视几个调优 kafka 消费端的常用选择。

在流式系统中，是非常值得考虑传输两端的性能的。如果你让你的生产端的效率提升了，相应的消费端也需要提升效率来适应。最起码不能让消费端成为性能瓶颈。正如我们接下来要介绍的内容一样，一些消费端的配置实际上依赖其生产段和 部分kafka 的通用配置。

## 如何观察 kafka 消费端的性能

在寻求优化您的消费者时，您当然希望控制在失败时消息会发生什么。 而且您还需要尽可能确保您已经适当地扩展了您的消费者组以处理您期望的吞吐量水平。 与生产者一样，您需要在开始进行调整之前监控消费者的表现。 从可靠性和稳定性方面考虑您对消费者的期望。

当我们试图调优消费端的时候，首先需要做的肯定是想要控制消息消费失败时做了什么。其次才需要尽可能的增加消费组的数量来让吞吐量尽可能的增大以适应业务。跟生产者一样，我们也需要手机消费端的性能指标数据用来比较调整参数之后的变化。可以先查阅 kafka 的 [Fetch Metrics](https://kafka.apache.org/documentation/#consumer_fetch_monitoring) 来确定我们需要检测的指标。

## 消费者基础配置

消费端的基础配置首先必须有 `host:port`形式的 bootstrap server 地址用来链接 kafka broker。
其次需要有传入一个 `deserializers` 策略来转换 消息keys 和 values。 `client.id` 可选，用来标记client

```bash
bootstrap.servers=localhost:9092
key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
client.id=my-client
```

## 关键配置
On top of our minimum configuration, there are a number of properties you can use to fine-tune your consumer configuration. We won’t cover all possible consumer configuration options here, but examine a curated set of properties that offer specific solutions to requirements that often need addressing:

- group.id
- fetch.max.wait.ms
- fetch.min.bytes
- fetch.max.bytes
- max.partition.fetch.bytes
- max.message.bytes (topic or broker)
- enable.auto.commit
- enable.commit.interval.ms
- isolation.level
- session.timeout.ms
- heartbeat.interval.ms
- auto.offset.reset
- group.instance.id
- max.poll.interval.ms
- max.poll.records

