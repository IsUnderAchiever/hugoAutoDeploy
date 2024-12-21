---
title: 12_注解方式配置AOP如何切换动态代理类型
description: 12_注解方式配置AOP如何切换动态代理类型
date: 2023-01-20
slug: 12_注解方式配置AOP如何切换动态代理类型
image: 202412212133331.png
categories:
    - Spring
---

## 12_注解方式配置AOP如何切换动态代理类型
> 有的时候我们需要修改AOP的代理方式。我们可以使用以下方式修改:
> 如果我们是采用注解方式配置AOP的话:
> 设置aop:aspectj-autoproxy标签的proxy-target-class属性为true，代理方式就会修改成Cglib
```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
```
> 如果我们是采用xml方式配置AOP的话:
> 设置aop:config标签的proxy-target-class属性为true,代理方式就会修改成Cglib
```xml
<aop:config proxy-target-class="true"/>
```
