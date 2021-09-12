---
title: Guava String笔记 (2) - Splitter
date: 2021-09-12 01:15:02
tags: guava
categories:
- 后端技术
- Google Guava
thumbnail:
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
本文继续简介一下Guava Splitter。


# Splitter
相关Core Java提供的String.split方式。Guava相对功能要靠谱许多。
Core Java中String.split方式有比较坑的地方。

请看如下示例，想一想结果应该是什么。
`",a ,,b,".split(",")`

A. `"", "a ", "", "b", ""`
B. `null, "a ", null, "b", null`
C. `"a ", null, "b"`
D. `"a ", "b"`


正确答案为`"", "a ", "", "b"`。

```java
@Test
void testCoreJavaSplit() {
    String sampleDate = ",a ,,b,";
    String[] actual = sampleData.split(",");
    String[] expected = new String[]{"", "a ", "", "b"};
    // assertJ
    assertThat(actual).usingRecursiveComparison().isEqualTo(expected);
}
```

说实话，我在看Guava文档之前，认为A是对的。可以看出String.split方法会跳过结尾空字符串。
同时String.split()方法返回的是String[]，不实用。

由此在实际开发过程中，我还是比较推荐使用Guava的Splitter

如果使用Splitter来处理上面CoreJava split对应示例，则会得到答案A
```java
@Test
void testGuavaSplitter() {
    String sampleDate = ",a ,,b,";
    List<String> actual = Splitter.on(",").splitToList(sampleDate);
    List<String> expected = Arrays.asList("", "a ", "", "b", "");
    assertThat(actual).usingRecursiveComparison().isEqualTo(expected);
}
```

## split方法
Guava Splitter提供了三个split方法，对不同返回类型。
1. `split(): Iterable<String>`
2. `splitToList(): List<String>`
3. `splitToStream(): Stream<String>`

`Iterable<String>`在开发中使用做后续处理没有那么方便
`List<String>`则相对容易处理，我比较推荐。后续示例也都用`splitToList()`
`Stream<String>`对应Lambda解决方案，是比较方便，但是由于`splitToStream`方法被标记为`@Beta`，所以现在来说，还是谨慎使用。


## 处理工具方法
### 跳过空字符串
```java
@Test
void testGuavaSplitter() {
    String sampleDate = ",a ,,b,";
    List<String> actual = Splitter.on(",")
            .omitEmptyStrings()
            .splitToList(sampleDate);
    List<String> expected = Arrays.asList("a ", "b");
    assertThat(actual).usingRecursiveComparison().isEqualTo(expected);
}
```
### Trim返回结果
#### Trim空格
```java
@Test
void testGuavaSplitter() {
    String sampleDate = ",a ,,b,";
    List<String> actual = Splitter.on(",")
            .omitEmptyStrings()
            .trimResults()
            .splitToList(sampleDate);
    List<String> expected = Arrays.asList("a", "b");
    assertThat(actual).usingRecursiveComparison().isEqualTo(expected);
}
```
#### 使用CharMatcher去除其它字符
```java
@Test
void testSplitterTrimResult() {
    String data = "alpha!|beta:|gamma;|omega_1|_sigma_";
    List<String> actual = Splitter
            .on("|")
            .trimResults(CharMatcher.anyOf("!:;_"))
            .splitToList(data);
    List<String> expected = Arrays.asList("alpha", "beta", "gamma", "omega_1", "sigma");
    assertThat(actual).usingRecursiveComparison().isEqualTo(expected);

}
```

## 限制分隔次数
```java
@Test
void testSplitterLimit() {
    String data = "alpha|beta|gamma|omega|sigma";
    List<String> actual = Splitter.on("|").limit(3).splitToList(data);
    List<String> expected = Arrays.asList("alpha", "beta", "gamma|omega|sigma");
    assertThat(actual).usingRecursiveComparison().isEqualTo(expected);
}
```

## 根据特写字符长度进行分隔
```java
@Test
void testSplitterFixedLength() {
    String data = "001002003911";
    List<String> actual = Splitter.fixedLength(3).splitToList(data);
    List<String> expected = Arrays.asList("001", "002", "003", "911");
    assertThat(actual).usingRecursiveComparison().isEqualTo(expected);
}
```

## CharMatcher匹配多个分隔符
```java
 @Test
void testCharMatcherSplitter() {
    String data = "alpha,beta;gamma_omega|sigma";
    List<String> actual = Splitter.on(CharMatcher.anyOf(",;_|")).splitToList(data);
    List<String> expected = Arrays.asList("alpha", "beta", "gamma", "omega", "sigma");
    assertThat(actual).usingRecursiveComparison().isEqualTo(expected);
}
```

## Split返回Map
此方法被标记为`@Beta`，生产谨慎使用。
```java
@Test
void testSplitterToMap() {
    String data = "language=java|version=1.8";
    Map<String, String> actual = Splitter
            .on("|")
            .withKeyValueSeparator("=")
            .split(data);
    Map<String, String> expected = Maps.newHashMap();
    expected.put("language", "java");
    expected.put("version", "1.8");
    assertThat(actual).usingRecursiveComparison().isEqualTo(expected);

    actual = Splitter
            .on("|")
            .withKeyValueSeparator(Splitter.on("="))
            .split(data);
    assertThat(actual).usingRecursiveComparison().isEqualTo(expected);
}
```

大家根据实际场景进行使用

Cheers~