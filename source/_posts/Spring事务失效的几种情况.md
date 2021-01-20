---
title: Spring事务失效的几种情况
date: 2021-01-20 19:57:57
tags: [Spring,事务]
categories: Spring
---

#### 一、数据库引擎

Mysql的MyISAM引擎不支持事务回滚，如果需要数据回滚，需要将Mysql的引擎设置为InnoDB;

#### 二、对异常进行捕获

Spring事务中默认的事务管理只能处理显示抛出的RuntimeException，如果对该异常进行try...catch处理，事务将不会生效；

#### 三、方法不为公共方法

使用@Transaction注解，该注解只针对public方法生效。同样需要指定出现那种异常的清下进行回滚：@Transactional(rollbackFor = Exception.class)

#### 四、方法内部调用

同类中事务的方法不能嵌套在其他方法中，Q类中A方法调用B方法，B方法开启事务注解，B方法中事务不会生效。

#### 五、事务覆盖

当类配置了@Transactional，方法也配置了@Transactional，方法的事务会覆盖类的事务配置信息。

#### 六、事务传播行为设置错误

@Transactional 注解属性 propagation（事务的传播行为） 设置错误

