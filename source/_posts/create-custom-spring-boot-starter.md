---
title: 创建自定义Spring Boot Starter
date: 2018-10-10 20:23:46
tags:
categories:
- 后端技术
- Spring Boot
thumbnail: http://markfredchen-blog.oss-cn-shanghai.aliyuncs.com/blog/blog-spring-boot.png
---
## 摘要
在大家使用Spring Boot进行开发的过程中，应该可以接触到Spring Boot提供的很多Starter。比如
- `spring-boot-starter-web`
- `spring-boot-starter-jdbc`
- ...

## Spring Boot AutoConfiguration原理
### Auto Configuration类
在Spring Boot启动的过程中，会去找`classpath:META-INF/spring.factories`文件，来决定加载哪些AutoConfiguration类。
让我们看一下spring-boot-autoconfigure项目中的`spring.factories`文件。下面列出部分AutoConfiguration类。
```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
...
```
Spring Boot会尝试去运行所有`org.springframework.boot.autoconfigure.EnableAutoConfiguration`列出的AutoConfiguration类。此处大家可以会有疑问，很多我在项目里没有用到。对应组件是不是也会被初始化？答案是不会。我们用`RedisAutoConfiguration`为例来看一下背后发生了什么。

```java
@Configuration
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")
	public RedisTemplate<Object, Object> redisTemplate(
			RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean
	public StringRedisTemplate stringRedisTemplate(
			RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

}
```
- Line 1: `@Configuration`定义Configuration组件
- Line 2: `@ConditionalOnClass(RedisOperations.class)` 在classpath中如果有`RedisOperation`类，则会运行此AutoConfiguration。反之则不会运行。
- Line 8: `@ConditionalOnMissingBean(name = "redisTemplate")` 如果在Spring容器中没有名为`redisTemplate`的Bean被注册，则执行redisTemplate()方法。反之则不会执行。

注：Spring Boot提供了一系列@Condition选择，同时还可以实现自定义Condition。参考[Spring Boot - Condition Annotations](https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/reference/htmlsingle/#boot-features-condition-annotations)

### AutoConfiguration Hints
AutoConfiguration Hints功能主要便于开发人员在做配置时，IDE可以提示auto complete。如下截图：
![auto-complete](https://markfredchen-blog.oss-cn-shanghai.aliyuncs.com/blog/blog-spring-boot-custom-starter-configuration-metadata.png)
Auto Complete信息包括：
- 配置名称
- 配置描述
- 配置默认值

具体信息在`additional-spring-configuration-metadata.json`中定义

## 创建自定义Starter
1. 需要封装的Service类
```java
public class FooService {
    private final String fooMessage;
    private final String barMessage;

    public FooService(String fooMessage, String barMessage) {
        this.fooMessage = fooMessage;
        this.barMessage = barMessage;
    }

    public String getFooMessage() {
        return this.fooMessage;
    }

    public String getBarMessage() {
        return this.barMessage;
    }
}
```
2. AutoConfiguration类
```java
package io.markfredchen.custom.starter.config;

import ...;

@Configuration
@PropertySource("classpath:config/foo.properties")
public class FooAutoConfiguration {
    @Value("${foo.fooMessage}")
    private String fooMessage;
    @Value("${foo.barMessage}")
    private String barMessage;

    @Bean
    @ConditionalOnMissingBean
    public FooService fooService() {
        return new FooService(fooMessage, barMessage);
    }
}

```
3. `spring.factories`
```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
io.markfredchen.custom.starter.config.FooAutoConfiguration
```
4. 默认配置
创建 `resources/config/foo.properties`
```properties
foo.fooMessage=Foo!
foo.barMessage=Bar!
```

## 使用自定义Starter
1. 创建新项目
2. SpringBootApplication
```java
@SpringBootApplication
public class FooApplication {
    public static void main(String[] args) {
        SpringApplication.run(FooApplication.class, args);
    }

    @Bean
    public CommandLineRunner runner(final FooService fooService) {
        return args -> {
            System.out.println(fooService.getFooMessage());
            System.out.println(fooService.getBarMessage());
        };
    }
}
```
3. `pom.xml`添加以下依赖
```xml
<dependencies>
    <dependency>
        <groupId>io.markfredchen</groupId>
        <artifactId>foo-spring-boot-starter</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.0.5.RELEASE</version>
    </dependency>
</dependencies>
```
4. 运行结果
```
2018-10-11 11:28:55.457  INFO 23978 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-10-11 11:28:55.457  INFO 23978 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-10-11 11:28:55.548  INFO 23978 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2018-10-11 11:28:55.586  INFO 23978 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2018-10-11 11:28:55.590  INFO 23978 --- [           main] i.m.c.s.foo.application.FooApplication   : Started FooApplication in 1.725 seconds (JVM running for 2.44)
Foo!
Bar!
```
5. 覆盖默认配置
创建`resources/application.properties`
```
foo.barMessage=Updated Bar!
```
6. 运行结果
```
2018-10-11 11:28:55.457  INFO 23978 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-10-11 11:28:55.457  INFO 23978 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-10-11 11:28:55.548  INFO 23978 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2018-10-11 11:28:55.586  INFO 23978 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2018-10-11 11:28:55.590  INFO 23978 --- [           main] i.m.c.s.foo.application.FooApplication   : Started FooApplication in 1.725 seconds (JVM running for 2.44)
Foo!
Updated Bar!
```
7. AutoConfiguration Hints
创建`resources/META-INF/additional-spring-configuration-metadata.json`
```
{
  "properties": [
    {
      "name": "foo.fooMessage",
      "type": "java.lang.String",
      "description": "Foo Message.",
      "defaultValue": "Foo!"
    },
    {
      "name": "foo.barMessage",
      "type": "java.lang.String",
      "description": "Bar Message.",
      "defaultValue": "Bar!"
    }
  ]
}
```
效果
![auto complete](https://markfredchen-blog.oss-cn-shanghai.aliyuncs.com/blog/blog-spring-boot-custom-starter-auto-complete.png)

## 参考
- [Spring Boot - Condition Annotations](https://docs.spring.io/spring-boot/docs/2.0.5.RELEASE/reference/htmlsingle/#boot-features-condition-annotations)

## 总结
本文介绍了spring boot starter机制以及如何创建自定义的starter。starter的目标是对现在有项目进行有效封装，减少开发人员的重复工作。
完整源代码访问[GitHub](https://github.com/markfredchen/tutorials/tree/master/spring-boot-customer-starter)
