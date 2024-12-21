---
title: 05_JoinPoint案例
description: 05_JoinPoint案例
date: 2023-01-16
slug: 05_JoinPoint案例
image: 202412212133331.png
categories:
    - Spring
---

## 05_JoinPoint案例
> 需求：在调用方法时，输出**`某某类中的某某方法被调用了，调用时传入的参数是哪些`**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
    <context:component-scan base-package="com.example"/>
    <context:property-placeholder location="classpath:project.properties"/>
    <aop:aspectj-autoproxy/>
</beans>
```
```properties
project.user.uid=2
project.user.username=李四
project.user.password=111111
project.phone.number=13195868989
project.phone.type=电信
```
```java
@Data
@Component
public class Phone {
    @Value("${project.phone.number}")
    private String number;
    @Value("${project.phone.type}")
    private String type;
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
```
```java
@Service
public class PhoneService {
    public void addPhone(Phone phone) {
        System.out.println("新增手机");
    }
    public void updatePhone(Phone phone) {
        System.out.println("修改手机");
    }
    public void deletePhone(String number) {
        System.out.println("删除手机");
    }
    public Phone getPhoneByNumber(String number) {
        System.out.println("查询手机");
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        return context.getBean(Phone.class);
    }
    public void testPhone(){
        System.out.println("测试手机");
    }
}
@Service
public class UserService {
    public void addUser(User user) {
        System.out.println("新增用户");
    }
    public void updateUser(User user) {
        System.out.println("修改用户");
    }
    public void deleteUser(Integer id) {
        System.out.println("删除用户");
    }
    public User getUserById(Integer id) {
        System.out.println("查询用户");
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        return context.getBean(User.class);
    }
    public void testUser(){
        System.out.println("测试用户");
    }
}
```
```java
@Component
@Aspect
public class MyAspect {
    @Pointcut("@annotation(com.example.aspect.InvokeLog)")
    public void logPointCut() {}
    @Pointcut("execution(* com.example.service..*.*(..))")
    public void servicePointCut() {}
    @Before("servicePointCut()")
    public void before(JoinPoint point){
        MethodSignature signature = (MethodSignature) point.getSignature();
        System.out.println("方法所在的全类名是:"+signature.getDeclaringTypeName());
        System.out.println("方法名称是:"+signature.getName());
        System.out.println("方法参数是:"+ Arrays.toString(point.getArgs()));
        System.out.println("-------------------------------");
    }
}
```
```java
@Component
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        // 对象
        User user = context.getBean(User.class);
        Phone phone = context.getBean(Phone.class);
        // service
        UserService userService = context.getBean(UserService.class);
        userService.addUser(user);
        userService.deleteUser(1);
        userService.updateUser(user);
        userService.getUserById(1);
        userService.testUser();
        PhoneService phoneService = context.getBean(PhoneService.class);
        phoneService.addPhone(phone);
        phoneService.deletePhone("1");
        phoneService.updatePhone(phone);
        phoneService.getPhoneByNumber("1");
        phoneService.testPhone();
    }
}
```
## @value获取不到值
>1. static的属性使用@Value注解，因为Spring的类加载器是在jvm之后的，所以在jvm加载过程中@Value不起作用
>2. 使用@Value注解的类被Spring所管理，如@Component、@Service、@Controller等
>3. new对象，Spring管理的对象，new出来，当然也获取不到
