---
title: 后端研发日志记录规范
date: 2020-07-01 17:00:00
categories: [笔记]
tags: [总结]     # TAG names should always be lowercase
---

## 背景

> 日志：记录程序的运行轨迹，方便查找关键信息，也方便快速定位解决问题。

随着公司发展，后端项目app的数量 越来越多，排查问题的复杂度越来越高，需要对日志的格式统一规划，便于后续日志收集分析报警。

## 日志的作用（WHY）
- **问题追踪：** 辅助排查和定位线上问题，优化程序运行性能。
- **状态监控：** 通过日志分析，可以监控系统的运行状态。
- **安全审计：** 审计主要体现在安全上，可以发现非授权的操作。

## 技术选型

经过调研，公司目前所有后端java 应用都是spring-boot 技术栈，而spring-boot的底层日志依赖关系如下图所示：
![图片](/assets/img/post/2019110810594598.png)

且，`logback`是 `slf4j`的默认实现，性能比`log4j`好很多,所以
1. 后端统一选择用`logback`作为日志记录依赖。
2. 使用 `lombok` 插件增强添加 `log` 变量


## 记录规范 (HOW)

### 记录时机
当符合以下情况时，需要选择合适的级别记录日志：
1. 无法处理的 `RuntimeException`,结合实际情况选择 warn 级别或 error级别
2. 执行流程不符合业务流程，比如：参数不正确，类型不正确和返回值不在预期范围内等。选择 Error 或 info 级别
3. 系统关键角色，组件核心动作： 比如服务之间的交互，数据库增删改等需要记录 info级别日志。
4. 常规初始化： 系统初始化的关键参数等，需要记录info级别日志。

### 如何记录（格式规范）

结合公司实际情况，日志记录应包含以下信息：
1. 日志时间（精确到毫秒）
2. 日志级别
3. 应用名称  - 项目名称
4. 调用链标识（可选）- 唯一字段
5. 业务标识
6. 线程名称 
7. 记录器名称（class名方法名）
8. 日志消息
9. 异常栈（可选）

### 日志文件名规范
当前正在写入的日志文件名：`<log-level>.log`

已经滚入历史的日志文件名：`<log_level>.log.<yyyy-MM-dd>`

### 禁止项
1. 禁止使用 System.out/error 记录
2. 禁止出现 e.printStackTrace()
3. 禁止线上环境 出现 debug 日志
4. 禁止循环中打印日志

## 日志配置文件和工具

### 配置文件
```xml

<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">

    <!--读取spring 配置-->
    <springProperty scope="context" name="logPath" source="logging.file.path" defaultValue="logs"/>
    <springProperty scope="context" name="appName" source="spring.application.name" defaultValue="ZL-PROJECT"/>
    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->

    <!--配置-->
    <property name="appName" value="${appName}"/>
    <!--系统环境变量配置-->
    <property name="log.outside.level" value="DEBUG"/>

    <property name="CONSOLE_LOG_PATTERN"
              value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%contextName][bizId:%X{bizId}][%thread] %-5level %logger{50} [trackId:%X{trackId}] - %msg%n" />

    <contextName>${appName}</contextName>
    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${CONSOLE_LOG_PATTERN}
            </pattern>
        </encoder>
    </appender>
    <!-- 按照每天生成日志文件 -->
    <appender name="ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>
            ${logPath}/error.log
        </file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${logPath}/bak/error.log.%d{yyyy-MM-dd}</FileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>${CONSOLE_LOG_PATTERN}-%caller{2}</pattern>
        </encoder>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>100MB</MaxFileSize>
        </triggeringPolicy>
        <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印错误日志 -->
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${logPath}/info.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${logPath}/bak/info.log.%d{yyyy-MM-dd}</FileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
        </encoder>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>100MB</MaxFileSize>
        </triggeringPolicy>

        <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印INFO日志 -->
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <appender name="DEBUG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>
            ${logPath}/debug.log
        </file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${logPath}/bak/debug.log.%d{yyyy-MM-dd}</FileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
        </encoder>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>100MB</MaxFileSize>
        </triggeringPolicy>

        <filter class="ch.qos.logback.classic.filter.LevelFilter"><!-- 只打印DEBUG日志 -->
            <level>DEBUG</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>


    <!-- 测试环境+开发环境. 多个使用逗号隔开. -->
    <springProfile name="test">
        <logger name="org.springframework.web" level="INFO"/>
        <logger name="com.jizhang.platform" level="INFO"/>
        <logger name="com.ibatis" level="${log.outside.level}"/>
        <logger name="com.ibatis.common.jdbc.SimpleDataSource" level="${log.outside.level}"/>
        <logger name="com.ibatis.common.jdbc.ScriptRunner" level="${log.outside.level}"/>
        <logger name="com.ibatis.sqlmap.engine.impl.SqlMapClientDelegate" level="${log.outside.level}"/>
        <logger name="java.sql.Connection" level="${log.outside.level}"/>
        <logger name="java.sql.Statement" level="${log.outside.level}"/>
        <logger name="java.sql.PreparedStatement" level="${log.outside.level}"/>
        <logger name="com.jizhang.platform.mapper" level="${log.outside.level}"/>
    </springProfile>

    <!-- 生产环境. -->
    <springProfile name="prod">
        <logger name="org.springframework.web" level="ERROR"/>
        <logger name="com.jizhang.platform" level="INFO"/>
        <logger name="com.ibatis" level="${log.outside.level}"/>
        <logger name="com.ibatis.common.jdbc.SimpleDataSource" level="${log.outside.level}"/>
        <logger name="com.ibatis.common.jdbc.ScriptRunner" level="${log.outside.level}"/>
        <logger name="com.ibatis.sqlmap.engine.impl.SqlMapClientDelegate" level="${log.outside.level}"/>
        <logger name="java.sql.Connection" level="${log.outside.level}"/>
        <logger name="java.sql.Statement" level="${log.outside.level}"/>
        <logger name="java.sql.PreparedStatement" level="${log.outside.level}"/>
        <logger name="com.jizhang.platform.mapper" level="${log.outside.level}"/>
    </springProfile>

    <!-- 日志输出级别 -->
    <root level="INFO">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="ERROR"/>
        <appender-ref ref="INFO"/>
        <appender-ref ref="DEBUG"/>
    </root>
</configuration>

```
### 配置文件使用


1. 将上述中的 logback-spring.xml 中的内容复制到项目 Resource 目录下，命名为： logback-spring.xml
2. 在 spring properties中指定以下配置
   ```
   logging.config=classpath:logback-spring.xml
   ```
3. 在 spring properties 配置文件中指定log 文件输出目录(绝对地址),默认为项目 jar包 执行目录，示例如下
   ```
   logging.file.path=/var/log/{appName}
   ```
4. 配置 appName  
在spring property 文件中指定`spring.application.name` 属性，logback 配置文件中配置了 `contextName`为 `spring.application.name`

4. 调用链表标识
   在需要 输出 调用链标识的时候 配置时，使用 `org.slf4j.MDC`定义`trackId`字段，具体可参考下面代码
   ```JAVA
   MDC.put("trackId","trackId");
   ```
    'trackId'： 可以认为是方法调用过程中的唯一不变的字段，比如：userId,或者操作的字段id,或者接到的请求跟踪id 等
5. 业务标识
   需要输出 业务标识字段的时候，使用`MDC`配置 `bizId`字段。

**注意**： MDC 的实现是通过 `ThreadLocal<Map<String,String>>`实现的,具体可是查阅 ch.qos.logback.classic.util.LogbackMDCAdapter 源码，所以要注意 MDC中字段的更新问题，在多线程环境中尤其要注意线程中的任务执行完毕的时候记得 调用 MDC.clear()方法清理。

如上配置可达到以下效果
```log
2020-06-01 15:34:19.630 [zl-platform][bizId:bizExample][main] INFO  com.jizhang.testlog.TestlogApplication [trackId:trackExample] - info message!!!
2020-06-01 15:34:19.630 [zl-platform][bizId:bizExample][main] ERROR com.jizhang.testlog.TestlogApplication [trackId:trackExample] - error message!!!!
2020-06-01 15:34:19.633 [zl-platform][bizId:bizExample][main] ERROR com.jizhang.testlog.TestlogApplication [trackId:trackExample] - java.lang.Exception: messages!!!,exception!
```



## 参考
- [Java日志记录最佳实践](https://www.jianshu.com/p/546e9aace657)
- [优秀日志实践准则](https://mp.weixin.qq.com/s/c3dTTkbTNqnZTy_Da19I6Q)


# Nginx 日志规范

## 日志格式
Nginx 日志格式统一为一下格式：
```
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"'
                      'rt=$request_time uct="$upstream_connect_time" uht="$upstream_header_time" urt="$upstream_response_time"';

```
以上格式在默认格式的基础上添加了反向代理的转发相关信息，供排查问题用。

## 目录格式
Nginx 默认日志目录为 /var/log/nginx， 反向代理的服务目录 日志目录无需设置，在默认目录下会有相关文件夹。

## 参考
- [Nginx 官方](https://docs.nginx.com/nginx/admin-guide/monitoring/logging/)
- [如何建设高吞吐量的日志平台](https://blog.qiniu.com/archives/8779)
- [Logstash Alternatives](https://sematext.com/blog//logstash-alternatives/)


## 日志收集
考虑到目前日志主要用于 ES 分析，暂定方案为 fileBeats 收集各个系统的日志（包括nginx），统一汇总到logstash 平台然后过滤分析到 ES搜索/kibana分析