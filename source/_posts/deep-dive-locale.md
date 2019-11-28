---
title: Locale深度解析
date: 2019-07-22 11:23:57
tags: core-java spring-boot
categories:
- 后端技术
- Core Java
thumbnail: https://markfredchen-blog.oss-cn-shanghai.aliyuncs.com/blog/20190722001-locale.jpg
description: Locale是日常开发中比较容易忽视的技术点。特别是开发一些只做国内市场，只有中文的项目时，Locale可能就直接被忽视了。而且在项目提出多语言支持的时候，因为没有很好的理解，可能给自己埋了很多坑。
---
# 摘要
Locale是日常开发中比较容易忽视的技术点。特别是开发一些只做国内市场，只有中文的项目时，Locale可能就直接被忽视了。而且在项目提出多语言支持的时候，因为没有很好的理解，可能给自己埋了很多坑。

# 什么是Locale
其实[`java.util.Locale`](https://docs.oracle.com/javase/10/docs/api/java/util/Locale.html)的Java Doc有很详细的解释，我就不过多解释。
> A Locale object represents a specific geographical, political, or cultural region. An operation that requires a Locale to perform its task is called locale-sensitive and uses the Locale to tailor information for the user. 

主要记住Locale实例包括以下信息就可以了。在多年的开发经验中，script和variant基本没有用到。就不过多介绍。
* language
* script
* country (region)
* variant


## lanugage
> ISO 639 alpha-2 or alpha-3 language code

在实际使用中，基本我们碰不到3位字母表示的语言。
## country
> ISO 3166 alpha-2 country code or UN M.49 numeric-3 area code.

同样实际使用中，基本使用2位字母表示的国家

## scirpt 和 variant

[IANA Language Subtag Registry](https://www.iana.org/assignments/language-subtag-registry/language-subtag-registry) 定义了完整列表。
感觉script是地区的别称，variant是方言。了解一下就好。

在实际使用中，大部分同学第一反应可能是会写出以下Locale。
- zh_CN
- en_US
- ja_JP

如果你也是只想到这些，请打开你的浏览器->开发者工具->控制台中输入以下js
```javascript
window.navigator.language
```
浏览器为中文，你又在国内。输出结果就是zh-CN
然后如果重新你设置浏览器语言，比如设置成英文。再执行一下，输出结果是en-CN
What?? en-CN?? 这是什么鬼？ 首先通过这个Locale，我们可以知道country，是和你实际在哪个地区有关。
那该怎么处理？ 后面详细说怎么应用。

# 应用场景
Locale的使用场景基本就是根据不同国家和语言，进行不同的显示。实际经验以下2点为主。
1. 多语言 (下文会详细说明)
2. 金额显示。
3. 日期格式显示。

以金额显示为例，如果没有类似开发经验的话，你可以为会想，金额还有不同的格式。一般不都是 1,000,000.00。那你就错了。举两个例子。
1. 日语。日本人金额是不带小数点的。
2. 法语。法语中千位分隔符为空格，小数点为逗号。比如 1 000 000,00

正确理解Locale，并正确使用可以写出既规范，又简练，质量又高的代码。而不是见招拆招，每个语言写一个自己的实现。

# 创建Locale 实例的正确姿势

**使用正确的姿势创建非常重要，这在后面Spring里应用部分非常重要。**

以下这段代码是我见过最多的创建方式。
```java
Locale locale = new Locale("zh_CN");
```
其中zh_CN可能是前端直接传入，为了方便直接作为Locale构造方法参数。其实这是一个错误的用法。这样的使用，创建出来的Locale.language就是zh_cn。

以下先列举一下两种正确姿势。然后比较一下结果
1. Locale API
```java
// 使用Locale构造方法
// 如果前端传入"zh_CN",此处需要自行解析并拆分
Locale locale = new Locale("zh", "CN");
// 使用Locale预置常量。请自行查看Locale源代码。
Locale locale = Locale.SIMPLIFIED_CHINESE
```
2. Commons-Lang `LocaleUtils.toLocale()`
```java
Locale locale = LocaleUtils.toLocale("zh_CN");
```

以上三种创建方式，可以创建出一样的Locale object。
本人推荐使用Commons Lang3 `LocaleUtils.toLocale()`


下面我们的来对比一下错误和正解方式创建的Locale有什么区别。
```java
public class LocaleShowCase {
    public static void main(String[] args) {
        logLocale(new Locale("zh_CN"));
        logLocale(Locale.SIMPLIFIED_CHINESE);
    }

    private static void logLocale(Locale locale) {
        System.out.println("=================================");
        System.out.println(String.format("Locale.toString: %s", locale.toString()));
        System.out.println(String.format("Language: %s", locale.getLanguage()));
        System.out.println(String.format("Country: %s", locale.getCountry()));
        System.out.println(String.format("LanguageTag: %s", locale.toLanguageTag()));
        System.out.println("=================================");
    }
}
```

**输出结果**

```log
=================================
Locale.toString: zh_cn
Language: zh_cn
Country: 
LanguageTag: und
=================================
=================================
Locale.toString: zh_CN
Language: zh
Country: CN
LanguageTag: zh-CN
=================================
```

让我们来分析一下结果
1. 首先看一下数据。错误的创建方式，其实是把zh_CN作为language。Country和LanguageTag为空
2. Language输出时均为小写
3. Country输出时均为大写
4. LanugageTag为Language和Country以"-"连接
5. Locale.toString则是Language和Country以"_"连接

## zh_CN vs zh-CN

什么时候使用"_"，什么时候使用"-"，确实比较搞。
比如request.getLocale()中解析Locale时，可以同时处理两种格式。而Commons Lang3 LocaleUtils.toLocale()的入参只支持下划线格式。

**不过我们可以定义这样的规范，在后端服务中只使用"_"格式，而前端只使用"-"格式。**


# 前端
前端框架太多，就只说一下最近在玩的umi+dva+react。
## UMI
### Locale处理
umi开发的项目中使用`umi-plugin-react/locale`来处理Locale。
```javascript
import { setLocale, getLocale } from 'umi-plugin-react/locale';

setLocale(language, true);
getLocale();
```
### 多语言
**资源文件使用"-"格式命名。**
```
.
|-- en-US
|   |-- common.ts
|   `-- form.ts
|-- ja-JP
|   |-- common.ts
|   `-- form.ts
|-- zh-CN
|   |-- common.ts
|   `-- form.ts
|-- en-US.ts
|-- ja-JP.ts
`-- zh-CN.ts
```
## 显示多语言
```javascript
import { formatMessage } from 'umi-plugin-react/locale';
formatMessage({id: 'xxx'})
```
## 日期显示
```javascript
import { formatDate } from 'umi-plugin-react/locale';
formatDate(new Date())；
```

## 数字显示
```javascript
formatNumber(10000000.00);
```


# 后端服务
这里只介绍基于Spring Boot开发的Stateless Rest API。SpringMVC已经过时，就不做介绍。

## Locale处理
### 如何确认当前API调用使用的Locale?
Spring Boot使用LocaleResolver来确定当前API调用使用什么Locale。在LocaleResolver获取Locale之后，将Locale存入LocaleContextHolder中。

Spring Boot提供了几个标准实现，主要区别是针对Locale存放的地方不一样提供对应获取方式
- AcceptHeaderLocaleResovler 从Request Header的Accept-Language中获取
- CookieLocaleResolver 从Cookie中获取
- FixedLocaleResolver 固定Locale，只使用系统配置的Locale
- SessionLocaleResolver 从Session中获取


虽然Spring已经提供了多种获取LocaleResolver实现，但是在具体业务场景中会有更复杂的场景。比如需要根据当前登录用户的语言设置。这个时间就需要我们自己实现一套LocaleResovler。

### 多语言
#### 资源文件
在Spring中，我们可以添加properties文件来做多语言支持。

```
.
|-- java
....
`-- resources
    `-- i18n
        |-- messages.properties
        |-- messages_ja.properties
        |-- messages_ja_JP.properties
        |-- messages_xx.properties
        |-- messages_zh.properties
        |-- messages_zh_CN.properties
        |-- another.properties
        |-- another_zh_CN.properties
        |-- another_zh_TW.properties
        |-- another_en.properties
        `-- another_ja.properties
```

可以看到在例子
- 有两组资源文件。一组为message，一组为anthor。在项目中可以这种方式进行模块化管理。
- 每组资源文件可以有自己支持的locale列表
- 每个文件定义对应locale的翻译
- 没有locale资源文件为默认语言。如messages.properties, another.properties。当没有locale匹配时，使用默认资源文件内容。
- messages_xx.properties? 这是合法的。但建议使用，也基本不会碰到。这里是说明一下，框架是支持的。用new Locale("xx")可以创建language为xx的Locale。

#### Locale匹配优先级
language+country+variant > language+country > lanaguage 

以message*.properties为例：
- zh_CN -> messages_zh_CN.properties
- zh或zh_JP -> messages_zh.properties
- en_US -> messages.properties.

#### 配置`MessageSource`
```java
@Configuration
public class MessageConfiguration {
    @Bean
    public MessageSource messageSource() {
        ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
        messageSource.setDefaultEncoding("UTF-8");
        messageSource.setBasenames("classpath:i18n/messages", "classpath:i18n/another");
        return messageSource;
    }
}
```
这里注意`messageSource.setBasenames("classpath:i18n/messages", "classpath:i18n/another")`BaseName就是资源文件的组名。

#### 使用方式
```java
@Service
public class XXXService {
    private final MessageSource messageSource;
    public XXXService(MessageSource messageSource) {
        this.messageSource = messageSource;
    }
    public String getI18N(String key, Object[] params) {
        return messageSource.getMessage(key, params, LocaleContextHolder.getLocale())
    }
}

```
## 日期显示
```java
DateFormat fullDF = DateFormat.getDateInstance(DateFormat.FULL, locale);
System.out.println(fullDF.format(new Date()));
```
## 数字显示
```java
System.out.println(NumberFormat.getInstance(locale).format(10000000));
```
# 实际场景中的应用

一般产品基本需要用户登录，在LocaleResovler中也提到。我们可以根据当前用户的语言设置作为使用Locale。这样比较好控制服务接收到的Locale。而且我们在开发时可以定义好系统支持的语言，比如支持zh_CN, en_US, ja_JP。这样在用户登录后的API调用就不用担心接收到不支持的Locale。而因为需要使用用户设置语言，我们需要自己实现一个LocaleResovler。

```java
@Data
public class Principal {
    private String username;
    private String language;
    ...
}
public class CustomLocaleResolver implements LocaleResolver {
    private Locale defaultLocale;

    public CustomLocaleResolver(Locale defaultLocale) {
        this.defaultLocale = defaultLocale;
    }

    public Locale resolveLocale(HttpServletRequest request) {
        Principal principal = (Principal) SecurityContextHolder.getContext().getAuthentication();
        if (principal != null && !StringUtils.isEmpty(principal.getLanguage())) {
            return LocaleUtils.toLocale(principal.getLanguage());
        } else {
            return request.getHeader("Accept-Language") != null ? request.getLocale() : this.defaultLocale;
        }
    }

    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
        throw new UnsupportedOperationException("Cannot change Principal data - use a different locale resolution strategy");
    }
}

@Configuration
public class LocaleConfiguration {
    @Bean
    public CustomLocaleResolver localeResolver(@Value("${default-language:zh_CN}") String defaultLanguage) {
        return new CustomLocaleResolver(LocaleUtils.toLocale(defaultLanguage));
    }
}

```

而用户没有登录之前，而前端不做任何处理时，后端会接收到类似en_CN的Locale，而无法匹配资源文件。如果按以下资源文件设计，给每个语言设置一个默认翻译，则可以解决接收到不规则Locale问题。

```
.
|-- java
....
`-- resources
    `-- i18n
        |-- messages.properties
        |-- messages_ja.properties
        |-- messages_ja_JP.properties
        |-- messages_en.properties
        |-- messages_en_US.properties
        |-- messages_en_GB.properties
        |-- messages_zh.properties
        |-- messages_zh_CN.properties
        `-- messages_zh_TW.properties
```
这样的编排方式，不管你在什么国家。对于中文，英文，日文都有一个默认的匹配。

Cheers~