---
title: 09_使用xml的方式配置AOP
description: 09_使用xml的方式配置AOP
date: 2023-01-19
slug: 09_使用xml的方式配置AOP
image: 202412212133331.png
categories:
    - Spring
---

## 09_使用xml的方式配置AOP
> 1. 定义切面类
> 2. 目标类和切面类注入容器
> 3. 配置AOP
```properties
project.user.uid=2
project.user.username=李四
project.user.password=111111
project.phone.number=13195868989
project.phone.type=电信
```
```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface InvokeLog {
}
@Component
public class MyAspect {
    @Pointcut("execution(* com.example.service..*.*(..))")
    public void servicePointCut() {
    }
    public void before(JoinPoint point){
        MethodSignature signature = (MethodSignature) point.getSignature();
        System.out.println("方法所在的全类名是:"+signature.getDeclaringTypeName());
        System.out.println("方法名称是:"+signature.getName());
        System.out.println("方法参数是:"+ Arrays.toString(point.getArgs()));
    }
    public void after(JoinPoint point){
        MethodSignature signature = (MethodSignature) point.getSignature();
        System.out.println("方法所在的全类名是:"+signature.getDeclaringTypeName());
        System.out.println("方法名称是:"+signature.getName());
        System.out.println("方法参数是:"+ Arrays.toString(point.getArgs()));
    }
    public void afterReturning(JoinPoint point,Object result){
        MethodSignature signature = (MethodSignature) point.getSignature();
        System.out.println("方法所在的全类名是:"+signature.getDeclaringTypeName());
        System.out.println("方法名称是:"+signature.getName());
        System.out.println("方法参数是:"+ Arrays.toString(point.getArgs()));
        System.out.println("result:"+result);
    }
    public void afterThrowing(JoinPoint point,Throwable throwable){
        System.out.println("方法的异常对象是:"+throwable);
    }
    public Object around(ProceedingJoinPoint point) {
        // 方法调用时传入的参数
        Object[] args = point.getArgs();
        // 被代理对象
        Object target = point.getTarget();
        // 获取被增强方法签名封装的对象
        MethodSignature signature = (MethodSignature) point.getSignature();
        Object proceed = null;
        try {
            proceed = point.proceed();
        } catch (Throwable e) {
            throw new RuntimeException(e);
        }
        return proceed;
    }
}
@Data
@Component
public class User {
    @Value("${project.user.uid}")
    private Integer uid;
    @Value("${project.user.username}")
    private String username;
    @Value("${project.user.password}")
    private String password;
}
@Service
public class UserService {
    @InvokeLog
    public void addUser(User user) {
        System.out.println("新增用户");
    }
    @InvokeLog
    public void updateUser(User user) {
        System.out.println("修改用户");
    }
    @InvokeLog
    public void deleteUser(Integer id) {
        System.out.println("删除用户");
    }
    @InvokeLog
    public User getUserById(Integer id) {
        System.out.println("查询用户");
        //List<Integer> num = null;
        //num.add(1);
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        return context.getBean(User.class);
    }
    @InvokeLog
    public void testUser(){
        System.out.println("测试用户");
    }
}
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService service = context.getBean(UserService.class);
        User userById = service.getUserById(1);
        // 这里userById为null的原因，是因为这是前置通知，目标方法还未执行，所以没有返回值
        System.out.println("返回值是:"+userById);
    }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
    <context:component-scan base-package="com.example"/>
    <context:property-placeholder location="classpath:project.properties"/>
    <!--配置AOP-->
    <aop:config>
        <!--定义切点-->
        <aop:pointcut id="pointCut" expression="@annotation(com.example.aspect.InvokeLog)"/>
        <aop:pointcut id="pointCut1" expression="execution(* com.example.service..*.*(..))"/>
        <!--配置通知方法-->
        <aop:aspect id="myAspect" ref="myAspect">
            <aop:before method="before" pointcut-ref="pointCut"/>
            <aop:after method="after" pointcut-ref="pointCut"/>
            <aop:after-returning method="afterReturning" pointcut-ref="pointCut" returning="result"/>
            <aop:after-throwing method="afterThrowing" pointcut-ref="pointCut" throwing="throwable"/>
        </aop:aspect>
    </aop:config>
</beans>
```
