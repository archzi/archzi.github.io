---
title: Java 标准库中的那些 Interface
date: 2020-06-01 17:00:00
categories: [Java]
tags: [[总结],[挖坑]]     # TAG names should always be lowercase
---


# 引言
天天使用 Java 编程，自然明白 `Interface` 对一个设计良好的应用的重要性。这边对标准库中的`Interface` 做一个简单的总结，希望对你有所帮助。

# java.lang
## Top leval 包
在java.lang 包下，没有子模块的包，我愿称之为java 顶级接口，是其他的基础。

### Appendable 
`Appendable` Interface 使得任意一个子类拥有如下 字符串 append 的方法
```java
Appendable append(CharSequence csq) throws IOException;

Appendable append(CharSequence csq, int start, int end) throws IOException;

Appendable append(char c) throws IOException;
```
需要注意的是 append 方法返回的是 对象本身，意味着我们 可以 `append("1").append("2")` 这样连续调用，常用的 `StringBuilder` 和 `StringBuffer` 就是其子类。

### AutoCloseable 

`AutoCloseable` Interface 使得任意一个子类 拥有 close() 方法，同时对 `AutoCloseable` 对象使用 `try-with-resource` 进行关闭。
疑问：为啥 AutoCloseable 类要放到 java.lang 包中，而不是跟 `Closeable` 对象一样放到 java.io 包中？猜测可能是为了适配 Stream 吧。


### CharSequence 

`CharSequence` Interface 用来描述一个 char 序列。

```java
int length();

char charAt(int index);

CharSequence subSequence(int start, int end);

public String toString();
```
同时接口也在 1.8 版本添加 stream 相关支持，默认实现了如下方法：
```java
public default IntStream chars(){...}

public default IntStream codePoints(){...}
```

### Cloneable
```java
public interface Cloneable {
}
```
`Cloneable` Interface 的源码如上，大家初次接触此类肯定回奇怪，为啥接口中什么都没有定义呢。
关于答案可以参考[知乎上 R 大的解释](https://www.zhihu.com/question/52490586/answer/130786763)

### Comparable
`Comparable` 对象标识两个对象可以比较大小 comparable_1 - comparable_2 的结果是 -1， 0  or 1
```java
public int compareTo(T o);
```


### Iterable 
`Iterable` 接口表示了 java 中一切可以`for-each loop`遍历的对象。
```java
Iterator<T> iterator(); // 第一个方法就返回 Iterator 对象

default void forEach(Consumer<? super T> action){...} // 默认实现 forEach 方法，接受一个 Consumer action

default Spliterator<T> spliterator() {...} // spliterator 用调用 iterator 方法来 支持 Spliterator
```

### Readable 

`Readable` 对象表示对象可以调用如下方法：意思是从 `Readable` 对象中 `read` 数据保存到 `CharBuffer` 中，返回读取的长度。
```java
public int read(java.nio.CharBuffer cb) throws IOException;
```

### Runnable 

`Runnable` 对象表示此对象可以调用如下方法，意思就是表示一段可执行的 java 代码。
```java
public abstract void run();
```

## annotation
java.lang.annotation 包中就一个接口表示 Java 中的 `Annotation`。

### Annotation


## instrument

## invoke

## management

## reflect