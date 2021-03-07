---
layout: post
title: springmvc 参数注入
categories: spring
tags: spring springmvc
author: leihtg

---

* content
{:toc}

WebMvcConfigurationSupport 中会加载参数格式化类 DefaultFormattingConversionService

默认启动方式  
BackgroundPreinitializer -> onApplicationEvent() -> performPreinitialization() -> run() 


DateTimeFormat -> JodaDateTimeFormatAnnotationFormatterFactory


org.springframework.web.servlet.DispatcherServlet#doDispatch  
由DispatcherServlet的doDispatch进入  
                
org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter#handle

org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter#handleInternal

org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter#invokeHandlerMethod

org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod#invokeAndHandle

org.springframework.web.method.support.InvocableHandlerMethod#invokeForRequest

org.springframework.web.method.support.InvocableHandlerMethod#getMethodArgumentValues  
这个地方是RequestMapping接口的参数  

org.springframework.web.method.support.HandlerMethodArgumentResolverComposite#resolveArgument  
逐个解析参数  
HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);  
使用不同的resolver解析

HandlerMethodArgumentResolver -> resolveArgument  

org.springframework.web.method.annotation.ModelAttributeMethodProcessor#resolveArgument  
在这个方法里面实现了参数值的注入  

org.springframework.web.servlet.mvc.method.annotation.ServletModelAttributeMethodProcessor#bindRequestParameters


org.springframework.web.bind.ServletRequestDataBinder#bind

org.springframework.web.bind.WebDataBinder#doBind

org.springframework.validation.DataBinder#doBind

org.springframework.validation.DataBinder#applyPropertyValues

org.springframework.beans.AbstractPropertyAccessor#setPropertyValues(org.springframework.beans.PropertyValues, boolean, boolean)


org.springframework.beans.AbstractNestablePropertyAccessor#setPropertyValue(org.springframework.beans.PropertyValue)

org.springframework.beans.AbstractNestablePropertyAccessor#setPropertyValue(org.springframework.beans.AbstractNestablePropertyAccessor.PropertyTokenHolder, org.springframework.beans.PropertyValue)

org.springframework.beans.AbstractNestablePropertyAccessor#processLocalProperty

org.springframework.beans.AbstractNestablePropertyAccessor#convertForProperty

org.springframework.beans.AbstractNestablePropertyAccessor#convertIfNecessary

org.springframework.beans.TypeConverterDelegate#convertIfNecessary(java.lang.String, java.lang.Object, java.lang.Object, java.lang.Class<T>, org.springframework.core.convert.TypeDescriptor)  
这个里面会对各种参数分别处理, 如果是加了格式转换注解的,`ConversionService conversionService = this.propertyEditorRegistry.getConversionService();`


org.springframework.core.convert.support.GenericConversionService#convert(java.lang.Object, org.springframework.core.convert.TypeDescriptor, org.springframework.core.convert.TypeDescriptor)

org.springframework.core.convert.support.ConversionUtils#invokeConverter

org.springframework.core.convert.support.ObjectToObjectConverter#convert 
这个里面实现了赋值, 如果参数上没有格式转换的注解(如: @DateTimeFormat(pattern = "yyyy-MM-dd")), 将会直接调用参数类型的构造方法,如  
```java
class Param{
    
    Date date;
    
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    Date jd;
}
// 如果传的参数为 date=2020-12-10, 则date=new Date("2020-12-10"); 
```



