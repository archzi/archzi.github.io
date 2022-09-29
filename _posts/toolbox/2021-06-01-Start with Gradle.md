---
title: Gradle 基础学习笔记
date: 2021-06-01 17:00:00
categories: [构建工具]
tags: [[笔记],[挖坑]]     # TAG names should always be lowercase
---




# 介绍

gralde 是一个 开源的自动化构建工具，设计的初衷就是为了足够灵活，来满足几乎所有软件的构建需求。并且具有以下的功能：
1. **高性能**   只编译需要编译的部分（因为 task 的输入和输出变动），甚至可以在不同的机器上利用 build cache（有共享 cache 的条件下）
2. **基于 JVM**    熟悉java 的人很轻松就能学会
3. **基于约定实现**    gradle 借鉴了 maven，并建立了基于约定实现的通用项目很容易就能够构建，比如 java 项目，并且 这些实现也可以很轻松就被自定义，只需要遵循实现约定
4. **可扩展性**  可以轻松扩展 gradle 来自定义 tasks 或者 build model ，例如： 安卓的构建过程就添加了很多新的构建类型和风格
5. **IDE suppoort**  很多主流 IDE 可以引入 gradle 项目以便与与之交互，并且 gradle 还可以根据不同的 IDE 产生不同的 IDE 支持配置，例如ES 的 ./gradlew idea 命令就可以产生 idea 的配置文件
6. **Insight**    build scan and share 可以让构建过程能分享给其他人，与其他人交流并发现问题


# 关键点
## gradle 是一个通用型构建工具，而不只是为了构建 Java application 

![](/assets/img/post/image2021-6-7_14-2-33.png)

## 核心模型基于 tasks

gradle 的构建模型是基于tasks 的 DAG， 意味这 构建的本质是 根据配置文件 将很多 task 基于依赖关系组合起来。 一旦 DAG 创建完成之后 gradle 会根据 DAG 来确定任务的执行先后顺序。

tasks 由以下三个组成：
- Action -- 此 task 需要做的实际操作，比如：copy 文件和编译源码
- inputs -- task 的输入，比如配置文件，环境变量等
- outputs -- task 需要创建的文件目录和需要修改的文件等等

以上的提到的三个元素都不是必须的，task 依赖于本身需要做什么操作。gradle 的标准task 就都没有 任何 action，标准task 仅仅是聚合起来成为一个 构建约定。

**需要注意的点是:** gradle 基于tasks的模型使得 gradle 的增量编译很稳定和准确，除非真的需要 清除所有构建中间文件，否则尽量不要 做 clean 动作
![](/assets/img/post/image2021-6-7_12-43-38.png)


##  Gradle 有几个固定的构建阶段
   1. Initialization   -- 设置build 环境，确定 哪些 project 需要加入进来
   2. Configuation  -- 根据用户想要运行的 task 装配和配置 build 的 task DAG ，然后确定哪些 task 需要运行和运行顺序
   3. Execution  -- 运行 task 

>以上几个阶段跟 maven 的phase 不一样，maven 利用 phase 将 build 分成不同的 阶段，更像 gradle 的 各种task，但是又缺少了灵活性

##  gradle 的扩展方式不止一种

想要 gradle 内置所有的构建逻辑几乎是不可能的，大部分的构建应用都需要许多自定义逻辑来处理

   1. 自定义 task type - 通过把 源码放进 buildSrc 文件夹中或者 plugin 的方式可以自定义 task type
   2. 自定义 task action  - 通过 doFist() 和 doLast() 方法
   3. 在 project 和 tasks 中自定义 额外的 properties 
   4. 自定义约定  - 通过 plugin 来自定义 约定 像 Java build 一样
   5. 自定义模型 - 大部分语言都会自定义模型来让构建更有效和更快捷

##  构建脚本操作 API

将 Gradle 的构建脚本视为可执行代码很容易，因为它们就是这样。 但这是一个实现细节：精心设计的构建脚本描述了构建软件所需的步骤，而不是这些步骤应该如何完成工作。 这是自定义任务类型和插件的工作。

有一种常见的误解，认为 Gradle 的强大功能和灵活性来自它的构建脚本是代码这一事实。 这与事实相去甚远。 正是底层模型和 API 提供了强大的功能。 正如我们在最佳实践中建议的那样，您应该避免在构建脚本中放置过多的命令式逻辑（如果有的话）


# 与 Maven 比较

## 编译性能方面
gradle 和 maven 两种构建系统在实现上有着本质的区别，正如上文所述，gradle 是基于task 的 DAG，各种task 根据不同的职能做不同的事情；然而 maven 是基于线性的固定 phase 的模型，在maven构建的过程中，goals 只是附加到 项目的 phase 上，goals 和 gradle 的task 相类似。

在性能方面：maven 和gradle 都可以并行编译，但是 gradle 可以在初次编译之后进行 增量编译，大大减少编译的时长。并且 gradle 还具有以下特点可以加快编译的速度：

- java class 级别的 增量式变异
- Java 项目的 Compile avoidance
- 增量子任务api 的应用
- 编译守护进程来加速编译

## 管理依赖方面
Gradle 和 Maven 都可以处理动态和传递依赖项，使用第三方依赖项缓存，并读取 POM 元数据格式。您还可以通过中央版本控制定义声明库版本并强制执行中央版本控制。两者都从他们的工件存储库下载传递依赖项。 Maven 有 Maven Central，而 Gradle 有 JCenter，您也可以定义自己的私人公司存储库。如果需要多个依赖项，Maven 可以同时下载它们。

然而，Gradle 在 API 和实现依赖以及本质上允许并发安全缓存方面胜出。它还保留存储库元数据以及缓存的依赖项，确保使用相同缓存的两个或多个项目不会相互覆盖，并且它具有基于校验和的缓存并且可以与存储库同步缓存。此外，Gradle 与 IVY 元数据兼容，允许您定义自定义规则来为动态依赖项指定版本，并解决版本冲突。这些在 Maven 上不可用。

同时，gradle 在管理依赖方面也有很多新功能可以使用：

- 兼容库的替换规则的使用
- ReplacedBy 规则的使用
- 更好的元数据解析
- 使用外部依赖动态替换项目依赖的能力，反之亦然


gradle 在兼容 IDE 方面也更加灵活，可以根据不同的 IDE 生成不同的配置文件，让用户选择更加方便。

**总结：**

就执行模型而言，两者都有任务组和描述。两者都使您能够仅构建指定的项目及其依赖项。然而，Gradle 有一个完全可配置的 DAG，而使用 Maven，一个目标只能附加到另一个目标。多个目标采用有序列表的形式。 Gradle 还允许任务排除、传递排除和任务依赖推断。 Gradle 还具有用于任务排序和终结器等的高级功能。
管理构建基础设施是 Gradle 的另一个强项，因为它使用接受自动配置的包装器，而对于 Maven，您需要有一个扩展来支持自配置构建。 Gradle 还使您能够配置基于版本的构建环境，而无需手动设置这些环境。并且 gradle 还允许自定义如何分发应用。


# Demo 项目

参照 [samples/sample_building_java_applications](https://docs.gradle.org/current/samples/sample_building_java_applications.html)


# 参考
- [gradle user guide](https://docs.gradle.org/current/userguide/what_is_gradle.html)
- [gradle vs maven](https://stackify.com/gradle-vs-maven/)
- [gradle vs maven 2](https://gradle.org/maven-vs-gradle/)