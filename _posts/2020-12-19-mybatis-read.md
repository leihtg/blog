---
layout: post
title: MyBatis框架原理
categories: orm
tags: mybatis
author: leihtg
---



[toc]

#### MyBatis简介

MyBatis 本是apache的一个[开源项目](https://baike.baidu.com/item/开源项目/3406069)iBatis, 2010年这个[项目](https://baike.baidu.com/item/项目/477803)由apache software foundation 迁移到了[google code](https://baike.baidu.com/item/google code/2346604)，并且改名为MyBatis 。2013年11月迁移到[Github](https://baike.baidu.com/item/Github/10145341)。

iBATIS一词来源于“internet”和“abatis”的组合，是一个基于Java的[持久层](https://baike.baidu.com/item/持久层/3584971)框架。iBATIS提供的持久层框架包括SQL Maps和Data Access Objects（DAOs）



本文的mybatis版本是3.5.6的。



#### MyBatis核心组件

1. **Configuration**   

   MyBatis所有的配置信息都保存在Configuration对象之中，配置文件中的大部分配置都会存储到该类中

2. **Executor**

   MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护

3. **SqlSession**

   作为MyBatis工作的主要顶层API，表示和数据库交互时的会话，完成必要数据库增删改查功能

4. **MappedStatement**

   MappedStatement维护一条<select|update|delete|insert>节点的封装

5. **SqlSource** 

   负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回

6. **BoundSql**

   表示动态生成的SQL语句以及相应的参数信息

7. **StatementHandler**

   封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数等

8. **ParameterHandler**

   负责对用户传递的参数转换成JDBC Statement 所对应的数据类型

9. **ResultSetHandler**

   负责将JDBC返回的ResultSet结果集对象转换成List类型的集合

10. **TypeHandler**

    负责java数据类型和jdbc数据类型(也可以说是数据表列类型)之间的映射和转换



![](C:\Users\leihtg\Desktop\20141028140852531.png)