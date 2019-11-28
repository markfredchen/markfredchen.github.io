---
title: Spring Boot中使用Liquibase最佳实践
date: 2018-10-09 17:45:43
tags:
- spring-boot
- liquibase
categories:
- 后端技术
- Spring Boot
thumbnail: http://markfredchen-blog.oss-cn-shanghai.aliyuncs.com/blog/20181009001-spring-boot-liquibase.png
description: Liquibase作为快速开发工具，没有很好的规范。项目大了之后，可就苦了。
---
## 背景

## Liquibase问题
随着项目的发展，一个项目中的代码量会非常庞大，同时数据库表也会错综复杂。如果一个项目使用了Liquibase对数据库结构进行管理，越来越多的问题会浮现出来。
1. ChangeSet文件同时多人在修改，自己的ChangeSet被改掉，甚至被删除掉。
2. 开发人员将ChangeSet添加到已经执行过的文件中，导致执行顺序出问题。
3. 开发人员擅自添加对业务数据的修改，其它环境无法执行并报错。
4. ChangeSet中SQL包含schema名称，导致其它环境schema名称变化时，ChangeSet报错。
5. 开发人员不小心改动了已经执行过的ChangeSet，在启动时会报错。


## Liquibase基本规范
1. ChangeSet id使用[任务ID]-[日期]-[序号]，如 T100-20181009-001
2. ChangeSet必须填写author
3. Liquibase禁止对业务数据进行sql操作
4. 使用`<sql>`时，禁止包含schema名称
5. Liquibase禁止使用存储过程
6. 所有表，列要加remarks进行注释
7. 已经执行过的ChangeSet严禁修改。
8. 不要随便升级项目liquibase版本，特别是大版本升级。不同版本ChangeSet MD5SUM的算法不一样。

其它数据库规范不再赘述。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">
    <changeSet id="T100-20181009-001" author="markfredchen" >
        <createTable tableName="demo_user" remarks="用户表">
            <column name="id" type="bigint" remarks="用户ID,主键">
                <constraints nullable="false" primaryKey="true" primaryKeyName="pk_demo_user_id"/>
            </column>
            <column name="username" type="varchar(100)" remarks="用户名">
                <constraints nullable="false"/>
            </column>
            ...
        </createTable>
    </changeSet>
</databaseChangeLog>

```

## 有效文件管理
使用Liquibase中提供`<include file="xxx"/>`tag，可以将ChangeSet分布在不同文件中。同时`<include/>`支持多级引用。
基于此功能可以对项目中的ChangeSet进行有效管理。推荐使用以下规范进行管理。
### 根据发布进行管理
1. 每个发布新建一个文件夹，所有发布相关的ChangeSet文件以及数据初始化文件，均放在些文件夹中。
2. 每个发布新建一个master.xml。此master.xml中，include本次发布需要执行的ChangeSet文件
3. 根据开发小组独立ChangeSet文件(可选)
4. 根据功能独立ChangeSet文件。例如user.xml, company.xml
```
resources
  |-liquibase
    |-user
    | |- master.xml
    | |- release.1.0.0
    | | |- release.xml
    | | |- user.xml -- 用户相关表ChangeSet
    | | |- user.csv -- 用户初始化数据
    | | |- company.xml -- 公司相关表ChangeSet
    | |- release.1.1.0
    | | |- release.xml
    | | |- ...
```

## 模块化管理
当项目变得庞大之后，一个服务可能包含的功能模块会越来越多。此时大家会想尽办法进行模块拆分，逐步进行微服务化。然而在面对错综复杂的Liquibase ChangeSet就会无从下手。
针对这种将来可能会面对的问题，项目初期就对Liquibase进行模块化管理，将在未来带来很大收益。
首先说明一下Spring Boot中Liquibase默认是如何执行以及执行结果。
1. 在启动时，LiquibaseAutoConfiguration会根据默认配置初始化SpringLiquibase
2. SpringLiquibase.afterPropertiesSet()中执行ChangeSet文件
3. 第一次跑ChangeSets的时候，会在数据库中自动创建两个表`databasechangelog`和`databasechangeloglock`

因此我们可以认为一个SpringLiquibase执行为一个模块。

引入多模块管理时，基于上节文件管理规范，我们基于模块管理再做下调整。
```
resources
  |-liquibase
    |-user
    | |- master.xml
    | |- release.1.0.0
    | | |- release.xml
    | | |- user.xml -- 用户相关表ChangeSet
    | | |- user.csv -- 用户初始化数据
    | | |- company.xml -- 公司相关表ChangeSet
    | |- release.1.1.0
    | | |- release.xml
    | | |- ...
    |- order
    | |- master.xml
    | |- release.1.0.0
    | | |- ...
```
当有一天我们需要把订单模块拆分成独立服务时，我们只需要将模块相关的ChangeSet文件迁出来。即可完成数据结构的拆分。

那如何在一个Spring Boot运行多个SpringLiquibase呢？需要对代码进行以下调整。

1. 禁用Spring Boot自动运行Liquibase。
当以下配置被启用时，Spring Boot AutoConfigure会使用默认配置初始化名为springLiquibase的Bean。然后我们不对其进行配置，Spring Boot启动时会报错。

```properties
# application.properties
# spring boot 2以上
spring.liquibase.enabled=false
# spring boot 2以下
liquibase.enabled=false
```
2. Spring Boot配置Liquibase Bean

配置两个SpringLiquibase Bean，Bean名称分别为userLiquibase和orderLiqubase。

```java
@Configuration
public class LiquibaseConfiguration() {

    /**
     *  用户模块Liquibase   
     */
    @Bean
    public SpringLiquibase userLiquibase(DataSource dataSource) {
        SpringLiquibase liquibase = new SpringLiquibase();
        // 用户模块Liquibase文件路径
        liquibase.setChangeLog("classpath:liquibase/user/master.xml");
        liquibase.setDataSource(dataSource);
        liquibase.setShouldRun(true);
        liquibase.setResourceLoader(new DefaultResourceLoader());
        // 覆盖Liquibase changelog表名
        liquibase.setDatabaseChangeLogTable("user_changelog_table");
        liquibase.setDatabaseChangeLogLockTable("user_changelog_lock_table");
        return liquibase;
    }
    /**
     *  订单模块Liquibase   
     */
    @Bean
    public SpringLiquibase orderLiquibase() {
      SpringLiquibase liquibase = new SpringLiquibase();
      liquibase.setChangeLog("classpath:liquibase/order/master.xml");
      liquibase.setDataSource(dataSource);
      liquibase.setShouldRun(true);
      liquibase.setResourceLoader(new DefaultResourceLoader());
      liquibase.setDatabaseChangeLogTable("order_changelog_table");
      liquibase.setDatabaseChangeLogLockTable("order_changelog_lock_table");
      return liquibase;
    }
}
```

Cheers~~
