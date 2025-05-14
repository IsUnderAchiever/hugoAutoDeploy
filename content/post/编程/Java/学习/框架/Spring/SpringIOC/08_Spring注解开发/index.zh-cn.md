---
title: 08_Spring注解开发
description: 08_Spring注解开发
date: 2023-01-15
slug: 08_Spring注解开发
image: 202412212133331.png
categories:
    - Spring
---

## Spring注解开发
```xml
<bean class="com.example.entity.User" id="userEntity"/>
```
等同于
```java
@Data
@Component("userEntity")
public class User {
    private Integer uid;
    private String username;
    private String password;
}
```
> 注解开发的准备工作
### 开启组件扫描
如果要使用注解开发必须要开启组件扫描，这样加了注解的类才会被识别出来。Spring才能去解析其中的注解。
```xml
<!--启动组件扫描，指定对应扫描的包路径，该包及其子包下所有的类都会被扫描，加载包含指定注解的类-->
<context:component-scan base-package="com.example"/>
```
------
> 这里试一下连接数据库，查数据
> 普通的jdbc的方法
```xml
<!-- applicationContext -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="com.example"/>
    <import resource="classpath:/mysql/jdbc.xml"/>
</beans>
```
```xml
<!-- jdbc.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    <context:property-placeholder location="classpath:mysql/jdbc.properties"/>
    <bean class="com.alibaba.druid.pool.DruidDataSource" id="dataSource">
        <property name="driverClassName" value="${spring.datasource.driver-class-name}"/>
        <property name="url" value="${spring.datasource.url}"/>
        <property name="username" value="${spring.datasource.username}"/>
        <property name="password" value="${spring.datasource.password}"/>
    </bean>
</beans>
```
```properties
spring.datasource.url= jdbc:mysql://localhost:3306/test?serverTimezone=UTC
spring.datasource.username= root
spring.datasource.password= 123456
spring.datasource.driver-class-name= com.mysql.cj.jdbc.Driver
spring.datasource.name= com.alibaba.druid.pool.DruidDataSource
```
```java
public interface UserDao {
    String selectUserById(Integer id);
    String updateUserStatusById(Integer id,String status);
}
@Repository("userDao")
public class UserDaoImpl implements UserDao {
    @Override
    public String selectUserById(Integer id) {
        return "select * from user where uid="+id;
    }
    @Override
    public String updateUserStatusById(Integer id,String status) {
        return "update user set status='"+status+"' where uid="+id;
    }
}
public class Application_10 {
    public static void main(String[] args) throws Exception {
        int id=1;
        String status="睡觉";
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        DruidDataSource dataSource = (DruidDataSource) context.getBean("dataSource");
        UserDao userDao= (UserDao) context.getBean("userDao");
        // 1.导入驱动jar包
        // 2.加载驱动
        Class.forName(dataSource.getDriverClassName());
        // 3.获取数据库连接对象
        Connection connection = DriverManager.getConnection(dataSource.getUrl(), dataSource.getUsername(), dataSource.getPassword());
        // 4.定义sql语句
        // 5.获取执行sql语句的对象
        Statement statement = connection.createStatement();
        // 6.执行sql并接收返回的对象
        statement.executeUpdate(userDao.updateUserStatusById(id,status));
        ResultSet resultSet = statement.executeQuery(userDao.selectUserById(id));
        // 7.处理结果
        while (resultSet.next()) {
            System.out.println("User(" + resultSet.getInt("uid") + ',' +
                    resultSet.getString("name") + ',' +
                    resultSet.getString("gender") + ',' +
                    resultSet.getInt("age") + ',' +
                    resultSet.getInt("parent_id") + ','+
                    resultSet.getString("status")+")");
        }
        // 8.关闭连接，释放资源
        resultSet.close();
        statement.close();
        connection.close();
    }
}
```
> 将uid为1的`user`的`status`字段更新为`睡觉`后查询输出
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201921812.png)
![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201922865.png)
> 常用注解有`@Component`、`@Repository`、`@Service`、`@Controller`
### @Value
> `@Value`并不依赖于`set方法`，支持SPEL表达式
```xml
<context:component-scan base-package="org.example"/>
```
```java
@ToString
@Component("userEntity")
public class User {
    @Value("1")
    private Integer uid;
    @Value("admin")
    private String username;
    @Value("123456")
    private String password;
}
```
```java
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        User userEntity = (User) context.getBean("userEntity");
        System.out.println(userEntity);
    }
}
```
> 我这里`User类`并没有给`set方法`，并不影响运行结果，说明`@Value`并不依赖于`set方法`
### @Autowired
![3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201922816.png)
```java
@Repository
public class UserDao {
    @Value("名字")
    private String name;
}
@Service("service")
public class UserService {
    @Autowired
    private UserDao userDao;
}
public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        //User userEntity = (User) context.getBean("userEntity");
        //System.out.println(userEntity);
        UserService userService = (UserService) context.getBean("service");
        System.out.println(userService);
    }
}
```
### @Qualifier
> 假如说，我们现在有多个`UserDaoImpl`实现`UserDao`
> 此时`@Autowired`根据类型会找到多个类，此时会发生报错
>
> `No qualifying bean of type 'org.example.dao.UserDao' available: expected single matching bean but found 2: userDao1,userDao2`
>
> 如何解决？
![4](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201922579.png)
```java
public interface UserDao {
}
@Repository("userDao1")
public class UserDaoImpl implements UserDao {
    @Value("名字")
    private String name;
}
@Repository("userDao2")
@ToString
public class UserDaoImpl2 implements UserDao {
    @Value("名字2")
    private String name;
}
```
![5](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201922963.png)
> 可以看到Idea这里也提示了报错信息
```java
@Service("userService")
@ToString
public class UserService {
    @Autowired
    @Qualifier("userDao1")
    private UserDao userDao;
}
```
可以通过`@Qualifier("BeanName")`来指定对应的bean
这里的`@Qualifier("userDao1")`和`UserDao`接口下的`@Repository("userDao1")`对应
> 注意，`@Qualifier`注解不能单独使用，需配合`@Autowired`来使用
