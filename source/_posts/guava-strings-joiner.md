---
title: Guava String笔记 (1) - Joiner
date: 2021-09-11 20:28:32
tags: guava
categories:
- 后端技术
- Google Guava
thumbnail:
# description: 本文主要介绍Guava Joiner
---
# 概要
Google Guava提供了如下几大类工具方法类。来简化对应代码逻辑。
- Joiner
- Splitter
- CharMatcher
- Charsets
- CaseFormat
- Strings

此系列笔记将从实际开发使用的角度，来看看Guava String工具类的优点。
本文先简介一下Guava Joiner。


# 前提条件
项目添加Guava依赖。最新版本查看[mvnrepository.com](https://mvnrepository.com/artifact/com.google.guava/guava)
## Maven
pom.xml
```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>30.1.1-jre</version>
</dependency>
```
## Gradle
build.gradle
```groovy
dependencies {
    implementation 'com.google.guava:guava:30.1.1-jre'
}
```



# Joiner

其实Guava Joiner和String.join目标一样，把一个集合转换成一个字符串。
以下代码等价

```java
@Test
public void testStringJoin() {
    List<Integer> data = Arrays.asList(1,2,null,3);
    String guavaJoinerResult = Joiner.on(",").join(data);
    assertEquals(guavaJoinerResult, "1,2,null,3");
    String stringJoinResult = String.join(",", data);
    assertEquals(stringJoinResult, "1,2,null,3");
}
```



以下是Guava Joiner额外提供的工具方法。如果有以下需求，就要使用Guava Joiner。

## null值处理
null值处理有两种方式。一是跳过，二是给定默认值

### null值跳过
```java
@Test
public void testStringJoinerSkipNull() {
    List<Integer> data = Arrays.asList(1,2,null,3);
    String actual = Joiner.on(",").skipNulls();
    assertEquals(actual, "1,2,3");
}
```

### null默认值
```java
@Test
public void testStringJoinerDefaultNullValue() {
    List<Integer> data = Arrays.asList(1,2,null,3);
    String actual = Joiner.on(",").useForNull("defaultValue");
    assertEquals(actual, "1,2,defaultValue,3");
}
```

## MapJoiner
处理Map，输出连接字符串

```java
@Test
public void testMapJoiner() {
    Joiner.MapJoiner joiner = Joiner.on(",").withKeyValueSeparator("=");
    Map<String, String> data = Maps.newHashMap();
    data.put("language", "Java");
    data.put("version", "1.8");
    assertEquals(joiner.join(data), "language=Java,version=1.8");
}
```

Cheers~