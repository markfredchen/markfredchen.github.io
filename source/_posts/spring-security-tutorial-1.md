---
title: 从零开始Spring Security (一)
date: 2018-10-11 18:19:45
tags: spring-boot spring-security
categories:
- 后端技术
- Spring Security
thumbnail: https://markfredchen-blog.oss-cn-shanghai.aliyuncs.com/blog/blog-spring-security-logo.png
description: 本文将一步步从零开始，搭建并讨论Spring Security原理
---
# 摘要
本文将一步步从零开始，搭建并讨论Spring Security原理
本文代码可以在 [GitHub](https://github.com/markfredchen/spring-security-demo) 中找到。
后续每篇文章相关代码会以分支的方式上传。


## 第一步
- Hello World Rest API
- 默认Spring Security保护

### Maven配置
```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>1.5.2.RELEASE</version>
	<relativePath/>
</parent>
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-security</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-devtools</artifactId>
		<scope>runtime</scope>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

### Java Code
```java
@SpringBootApplication
public class SpringSecurityDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringSecurityDemoApplication.class, args);
	}
}

@RestController
@RequestMapping("/api")
class DemoRestController {
	@GetMapping("/hello")
	public String greeting() {
		return "Hello World";
	}
}
```

### Demo
好了， 如此简单的代码。一个最基本的Rest API `/api/hello`已经开发完成，并被Spring Security通过Basic Auth进行保护。
```bash
➜  ~ curl http://localhost:8080/api/hello
{"timestamp":1489887539548,"status":401,"error":"Unauthorized","message":"Full authentication is required to access this resource","path":"/api/hello"}

➜  ~ curl http://user:6a3810c9-1ad7-4694-a617-0411fec20c60@localhost:8080/api/hello
Hello World
```

用户名密码在服务的启动Log中找。


```log
2017-03-19 09:30:31.492  INFO 76011 --- [  restartedMain] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-03-19 09:30:31.571  INFO 76011 --- [  restartedMain] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-03-19 09:30:31.959  INFO 76011 --- [  restartedMain] b.a.s.AuthenticationManagerConfiguration :

Using default security password: 6a3810c9-1ad7-4694-a617-0411fec20c60

2017-03-19 09:30:32.047  INFO 76011 --- [  restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Creating filter chain: OrRequestMatcher [requestMatchers=[Ant [pattern='/css/**'], Ant [pattern='/js/**'], Ant [pattern='/images/**'], Ant [pattern='/webjars/**'], Ant [pattern='/**/favicon.ico'], Ant [pattern='/error']]], []
2017-03-19 09:30:32.234  INFO 76011 --- [  restartedMain] o.s.s.web.DefaultSecurityFilterChain     : Creating filter chain: Ant [pattern='/h2-console/**'], [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@e89b69d, org.springframework.security.web.context.SecurityContextPersistenceFilter@1932f51e, org.springframework.security.web.header.HeaderWriterFilter@65c2ab9f, org.springframework.security.web.authentication.logout.LogoutFilter@3d247454, org.springframework.security.web.authentication.www.BasicAuthenticationFilter@1e897c55, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@75a44063, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@31201444, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@160c32f9, org.springframework.security.web.session.SessionManagementFilter@507dd201, org.springframework.security.web.access.ExceptionTranslationFilter@21062771, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@1f8069f6]
```

### 知识点
* `spring-boot-starter-security` 加上此依赖之后，Spring Boot会对app对一些最简单的安全配置。
* 默认Basic Auth的用户密码需要在app启动的日志寻找。

## 第二步

* 使用InMemoryAuthentication自定义用户访问
* API权限控制
* API中获取当前用户信息

### 添加WebSecurityConfiguration

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("admin")
                .password("admin-password")
                .roles("ADMIN")
                .and()
                .withUser("guest")
                .password("guest-password")
                .roles("GUEST");

    }
}
```
### SecurityUtils
```java
public class SecurityUtils {
    public static String getCurrentUser() {
        SecurityContext securityContext = SecurityContextHolder.getContext();
        Authentication authentication = securityContext.getAuthentication();
        if (authentication != null && authentication.getPrincipal() instanceof User) {
            return ((User) authentication.getPrincipal()).getUsername();
        } else {
            return null;
        }
    }
}
```
### API接口定义访问权限
```java
@RestController
@RequestMapping("/api")
class DemoRestController {
	@GetMapping("/hello")
	@PreAuthorize("hasRole('ADMIN')")
	public String greeting() {
		return "Hello " + SecurityUtils.getCurrentUser();
	}
}
```

### Demo
```bash
➜  ~ curl http://admin:admin-password@localhost:8080/api/hello
Hello admin
➜  ~ curl http://guest:guest-password@localhost:8080/api/hello
{"timestamp":1489904173901,"status":403,"error":"Forbidden","exception":"org.springframework.security.access.AccessDeniedException","message":"不允许访问","path":"/api/hello"}
```

### 知识点

* 当配置`EnableGlobalMethodSecurity(prePostEnabled = true)`时，Spring Security会启用方法权限控制。
* `@PreAuthorize("hasRole('ADMIN')")`方法权限控制。`@PreAuthorize`使用Spring Expression Language来描述方法权限。本例子为当用户有`ADMIN`权限时，才能访问。
* 当用户完成登录之后，默认设置下用户信息会以`org.springframework.security.core.userdetails.User`保存在`SecurityContextHolder`中

## 第三步

* 添加一个公共API，此API不需要进行Security保护。例如用户注册。

### WebSecurityConfiguration
在WebSecurityConfiguration中添加以下方法。
```java
@Override
public void configure(WebSecurity web) throws Exception {
    web.ignoring()
            .antMatchers("/api/public/api");
}
```

### DemoRestController
```java
@RestController
@RequestMapping("/api")
class DemoRestController {
	@GetMapping("/hello")
	@PreAuthorize("hasRole('ADMIN')")
	public String greeting() {
		return "Hello " + SecurityUtils.getCurrentUser();
	}

	@GetMapping("/public/api")
	public String publicAPI() {
		return "this is public API";
	}
}
```

### Demo

```bash
➜  ~ curl http://localhost:8080/api/public/api
this is public API
```

Cheers~