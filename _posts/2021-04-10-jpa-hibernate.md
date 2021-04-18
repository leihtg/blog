---
layout: post
title: spring集成hibernate-jpa
categories: jpa
tags: jpa
author: leihtg
---



* content
{:toc}


#### 引入
1. 使用`@EnableJpaRepositories`注解，可以配置`basePackages`来指定扫描的Dao
2. 不显式指定使用自动配置`JpaRepositoriesAutoConfiguration`

#### 加载过程

如果使用的是`@EnableJpaRepositories`则从`JpaRepositoriesRegistrar`进入，如果使用`JpaRepositoriesAutoConfiguration`则会从`JpaRepositoriesAutoConfigureRegistrar`进入.  
最终都会进入到JpaRepositoriesRegistrar中。接下来会比较重要的类`RepositoryConfigurationDelegate`，在方法`registerRepositoriesIn`中，
解析了注解`EnableJpaRepositories`所配置的参数，会把继承接口`JpaRepository`的所有自定义的接口分别注册到bean工厂中。
```java
    //build中配置需要的参数,比较重要的是repositoryFactoryBeanClass,这个在后面代理的时候要用
    BeanDefinitionBuilder definitionBuilder = builder.build(configuration);
```
加入到bean工厂后会等待初始化。初始化后会调用方法`JpaRepositoryFactoryBean`的afterPropertiesSet方法。
然后会给每个自定义的接口生成一个代理类`JdkDynamicAopProxy`,代理类的默认实现类是`SimpleJpaRepository`,
在实际查询中会调用方法链,里面方法嵌套很多，事务也是在这里面处理的。