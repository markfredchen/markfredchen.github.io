---
title: 分布式链路跟踪 Sleuth 与 Zipkin简易教程
date: 2019-11-18 11:40:27
tags:
categories:
thumbnail:
---

# 摘要
随着微服务的普及的同时，日志查询也不像以前单体那么简单。分页式链路跟踪也成为微服务框架必要的需求。Sleuth和Zipkin很好的解决了这个问题。
Sleuth和Zipkin的关系是。Sleuth负责标识链路，Zipkin负责采集日志并分析。 Sleuth是可以脱离Zipkin单独使用的。

# Get Started

创建两个Restful Service Application
- ZipkinBarServiceApplication
- ZipkinFooServiceApplication

分别提供一个Restful API
- foo-service/api/foo/{messge}
- bar-service/api/bar/{message}

## 创建两个service

### Bar Service
```java
@SpringBootApplication
public class ZipkinBarServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZipkinBarServiceApplication.class);
    }
}
```
org.opentest4j.AssertionFailedError: expected: tech.markfredchen.persistence.mybatis.plus.entity.User@74b8df0e
<User(id=1202802937379553281, username=update_user, phone=test_phone, isEnabled=true, isDeleted=false, createdDate=2019-12-06T12:12:21.928+08:00[Asia/Shanghai])> 
<User(id=1202802937379553281, username=update_user, phone=test_phone, isEnabled=true, isDeleted=false, createdDate=2019-12-06T12:12:21.928+08:00[Asia/Shanghai])>
	at org.junit.jupiter.api.AssertionUtils.fail(AssertionUtils.java:55)
	at org.junit.jupiter.api.AssertSame.failNotSame(AssertSame.java:48)
	at org.junit.jupiter.api.AssertSame.assertSame(AssertSame.java:37)
	at org.junit.jupiter.api.AssertSame.assertSame(AssertSame.java:32)
	at org.junit.jupiter.api.Assertions.assertSame(Assertions.java:2779)

