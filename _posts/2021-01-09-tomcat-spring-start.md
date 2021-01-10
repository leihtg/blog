---
layout: post
title: tomcat+springboot启动流程
categories: tomcat
tags: tomcat
author: leihtg
---



[toc]

原始入口  
org.apache.catalina.core.StandardContext#startInternal
```
            // Call ServletContainerInitializers
            for (Map.Entry<ServletContainerInitializer, Set<Class<?>>> entry :
                initializers.entrySet()) {
                try {
                    //所有实现ServletContainerInitializer接口的类
                    entry.getKey().onStartup(entry.getValue(),
                            getServletContext());
                } catch (ServletException e) {
                    log.error(sm.getString("standardContext.sciFail"), e);
                    ok = false;
                    break;
                }
            }

```


首先继承tomcat提供的容器初始化接口`ServletContainerInitializer`

```java
public class SpringServletContainerInitializer implements ServletContainerInitializer {
    public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
    			throws ServletException {
        //.....
        		for (WebApplicationInitializer initializer : initializers) {
        		    //初始化开始
        			initializer.onStartup(servletContext);
        		}
    			}
}
```
调用栈  
org.springframework.boot.web.servlet.support.SpringBootServletInitializer#onStartup

org.springframework.boot.web.servlet.support.SpringBootServletInitializer#createRootApplicationContext

org.springframework.boot.web.servlet.support.SpringBootServletInitializer#run

开始进入springboot启动流程  
org.springframework.boot.SpringApplication#run(java.lang.String...)

org.springframework.boot.SpringApplication#refreshContext

org.springframework.boot.SpringApplication#refresh

org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext#refresh

org.springframework.context.support.AbstractApplicationContext#refresh

org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext#onRefresh

创建web容器  
org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext#createWebServer

org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext#selfInitialize

这个方法里面开始向tomcat注入Servlet、Filter、EventListener等bean    
```
		for (ServletContextInitializer beans : getServletContextInitializerBeans()) {
		    //调用初始化
			beans.onStartup(servletContext);
		}
```
分别调用对应的`FilterRegistrationBean`,`ServletListenerRegistrationBean`,`ServletRegistrationBean`等的onStartup方法
