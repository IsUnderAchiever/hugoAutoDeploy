---
title: 09_XML配置文件相关注解
description: 09_XML配置文件相关注解
date: 2023-01-15
slug: 09_XML配置文件相关注解
image: 202412212133331.png
categories:
    - Spring
---

## XML配置文件相关注解
### @Configuration
> 标注在类上，表示当前类是一个`配置类`。我们可以用注解类来完全替换掉xml配置文件。
> `注意`:如果使用配置类替换了xml配置，`spring容器`要使用:`AnnotationConfigApplicationContext`
### @ComponentScan
> 可以用来代替contexcomponent-scan标签来配置组件扫描。
> 使用接下来的一个例子，来完全替换掉xml配置文件
```java
// java配置类
@Configuration
@ComponentScan(basePackages = {"com.example"})
public class ApplicationConfig {
}
@Data
@Component("userEntity")
public class User {
    @Value("1")
    private Integer uid;
    @Value("admin")
    private String username;
    @Value("123456")
    private String password;
}
public class Application_11 {
    public static void main(String[] args) {
        // 创建注解容器，指定配置类(传入字节码对象)
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class);
        User userEntity = (User) context.getBean("userEntity");
        System.out.println(userEntity);
    }
}
```
> `注意`：这里使用`AnnotationConfigApplicationContext`而不是之前的`ClassPathXmlApplicationContext`
### @Bean
> 可以用来代替bean标签，主要用于`第三方类`的注入。
> 比如Druid连接池，由于是第三方jar包，没办法添加`@Component`注解注入到`Spring容器`中
> 这时候，可以使用@Bean注解
```java
@Configuration
@ComponentScan(basePackages = {"com.example"})
public class ApplicationConfig {
    @Bean("dataSource")
    public DruidDataSource getDruidDataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("1");
        dataSource.setUrl("2");
        dataSource.setUsername("3");
        dataSource.setPassword("4");
        return dataSource;
    }
}
public class Application_11 {
    public static void main(String[] args) {
        // 创建注解容器，指定配置类(传入字节码对象)
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class);
        //User userEntity = (User) context.getBean("userEntity");
        //System.out.println(userEntity);
        DruidDataSource dataSource = (DruidDataSource) context.getBean("dataSource");
        System.out.println(dataSource);
    }
}
```
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201922130.png)
> 这里其实还有一种`更常用的写法`，不配置Bean的id，直接通过字节码来获取Bean
> 这里能这么使用是因为对应类型的Bean只有一个
![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201922749.png)
```java
@Configuration
@ComponentScan(basePackages = {"com.example"})
public class ApplicationConfig {
	// 没有指定Bean的id属性
    @Bean
    public DruidDataSource getDruidDataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("1");
        dataSource.setUrl("2");
        dataSource.setUsername("3");
        dataSource.setPassword("4");
        return dataSource;
    }
}
public class Application_11 {
    public static void main(String[] args) {
        // 创建注解容器，指定配置类(传入字节码对象)
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class);
        // 这里写DruidDataSource.class或者DataSource.class都行
        // 这里和config配置中的返回值类型相对应
        DataSource bean = context.getBean(DruidDataSource.class);
        System.out.println(bean);
    }
}
```
### @PropertySource
> 可以用来代替context:property-placeholder，让`Spring读取指定的properties文件`。然后可以使用@Value来获取读取到的值。
```properties
mysql.driverClassName=com.mysql.cj.jdbc.Driver
mysql.url=jdbc:mysql://localhost/test?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
mysql.username=root
mysql.password=123456
```
```java
@Configuration
@ComponentScan(basePackages = {"com.example"})
@PropertySource("classpath:jdbc.properties")
public class ApplicationConfig {
    @Value("${mysql.driverClassName}")
    private String driverClassName;
    @Value("${mysql.url}")
    private String url;
    @Value("${mysql.username}")
    private String username;
    @Value("${mysql.password}")
    private String password;
    @Bean
    public DruidDataSource getDruidDataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(driverClassName);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }
}
```
