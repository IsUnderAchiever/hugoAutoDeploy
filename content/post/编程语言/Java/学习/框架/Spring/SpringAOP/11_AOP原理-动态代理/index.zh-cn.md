---
title: 11_AOP原理-动态代理
description: 11_AOP原理-动态代理
date: 2023-01-20
slug: 11_AOP原理-动态代理
image: 202412212133331.png
categories:
    - Spring
---

## 11_AOP原理-动态代理
> 实际上Spring的AOP其实底层就是使用动态代理来完成的。并且使用了两种动态代理分别是JDK的动态代理和Cglib动态代理。
> 所以我们接下去来学习下这两种动态代理，理解下它们的不同点。
## JDK动态代理
> JDK的动态代理使用的java.lang.reflect.Proxy这个类来进行实现的。要求被代理（被增强)的类需要实现了接口。并且JDK动态代理也只能对接口中的方法进行增强。
```java
public class Main {
    public static void main(String[] args) {
        AIControllerImpl aiController = new AIControllerImpl();
        //String answer = aiController.getAnswer("张三很帅吗？");
        //System.out.println(answer);
        // 使用动态代理增强getAnswer方法
        // 1.JDK动态代理
        // 获取类加载器
        ClassLoader classLoader = Main.class.getClassLoader();
        // 被代理类所实现接口的字节码对象数组
        Class<?>[] interfaces = AIControllerImpl.class.getInterfaces();
        // 使用代理对象（proxy）时，会调用invoke
        AIController proxy = (AIController) Proxy.newProxyInstance(classLoader, interfaces, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                // proxy是代理对象
                // method是当前被调用的方法封装的Method对象
                // args是调用方法时传入的参数
                // 调用被代理对象的对应方法
                // 判断当前调用的是否是getAnswer方法
                if ("getAnswer".equals(method.getName())) {
                    System.out.println("已增强");
                }
                Object ret = method.invoke(aiController, args);
                return ret;
            }
        });
        String answer1 = proxy.getAnswer("张三很帅？");
        System.out.println(answer1);
    }
}
```
## Cglib动态代理
> 使用的是`org.springframework.cglib.proxy.Enhancer`类进行实现的。
> 已被spring中包含，pom导入ioc依赖即可
```java
public class Demo {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        // 设置父类的字节码对象。即enhancer生成的代理对象为AIControllerImpl的子对象
        enhancer.setSuperclass(AIControllerImpl.class);
        // 设置回调函数
        enhancer.setCallback(new MethodInterceptor() {
            // 使用代理对象执行方法时，都会调用到intercept方法中
            @Override
            public Object intercept(Object o, Method method, Object[] objects /*传入参数的数组*/, MethodProxy methodProxy) throws Throwable {
                // 判断当前调用的方法是不是getAnswer,若是，则进行增强
                if("getAnswer".equals(method.getName())) {
                    System.out.println("方法已增强");
                }
                // methodProxy.invokeSuper(); // 调用父类中对应的方法
                return methodProxy.invokeSuper(o, objects);
            }
        });
        // 生成代理对象
        AIControllerImpl proxy = (AIControllerImpl) enhancer.create();
        System.out.println(proxy.getAnswer("我帅吗"));
    }
}
```
## 总结
> JDK动态代理要求被代理（被增强）的类必须要实现接口，生成的代理对象相当于是被代理对象的兄弟。
>
> Cglib的动态代理不要求被代理（被增强）的类要实现接口，生成的代理对象相当于被代理对象的子类对象。
>
> Spring的AOP默认情况下优先使用的是JDK的动态代理，如果使用不了JDK的动态代理才会使用Cglib的动态代理。
