---
title: 02_Spring连接oracle
description: 02_Spring连接oracle
date: 2023-03-11
slug: 02_Spring连接oracle
image: 202412212036798.png
categories:
    - Java
---

## Spring连接oracle
**`使用过的jar包在文章结尾处，而且建议还是使用maven项目的结构，我这个结构不标准`**
> 新建lib包，在lib下导入jar包
>
> 如果发现无法使用jar包注解等，请在project structure->modules->dependencies中添加lib包
![image-20230311103940854](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303111039929.png)
> 这个是新建maven项目导入spring-context依赖后的图片
![image-20230311104018813](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303111040847.png)
> 这个是普通java项目手动导入jar包
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303111046042.png" alt="image-20230311104637001" style="zoom:80%;" />
![image-20230311104205883](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303111042953.png)
> 此时已经可以新建spring-xml了
### 测试lombok
```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer uid;
    private String name;
    private String gender;
    private Integer age;
}
```
```java
public class ApplicationTest {
    public static void main(String[] args) {
        User user = new User(1, "马小跳", "男", 15);
        System.out.println(user);
    }
}
```
> 测试成功
### 测试Spring
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="src.com.example.domain.User">
        <property name="uid" value="2"/>
        <property name="name" value="马小跳"/>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
    </bean>
</beans>
```
```java
public class ApplicationTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("src/applicationContext.xml");
        User user = (User) context.getBean("user");
        System.out.println(user);
    }
}
```
> 运行后报错，继续导入该jar包
![image-20230311104829411](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303111048473.png)
> 导入jar包后继续运行
![image-20230311110002275](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303111100403.png)
> 这里需要注意applicationContext.xml的位置
> 如果在src下，需要写`src/applicationContext.xml`
> 如果和src同级，需要写`applicationContext.xml`
## 配置Oracle
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303112242424.png" alt="image-20230311224226347" style="zoom:80%;" />
```properties
# 数据库驱动：
spring.datasource.driver-class-name=oracle.jdbc.driver.OracleDriver
# 数据库连接地址
spring.datasource.url=jdbc:oracle:thin:@localhost:1521:orcl
# 数据库用户名&密码：
spring.datasource.username=root
spring.datasource.password=123456
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    <context:property-placeholder location="classpath:src/jdbc.properties"/>
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${spring.datasource.driver-class-name}"/>
        <property name="url" value="${spring.datasource.url}"/>
        <property name="username" value="${spring.datasource.username}"/>
        <property name="password" value="${spring.datasource.password}"/>
    </bean>
</beans>
```
```java
public class ApplicationTest {
    @Test
    public void test1(){
        ApplicationContext context = new ClassPathXmlApplicationContext("src/applicationContext.xml");
        DruidDataSource dataSource = context.getBean("dataSource", DruidDataSource.class);
    }
}
```
![image-20230311224204289](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303112242465.png)
> 可以看到已经配置成功
>
> 接下来测试一下sql
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer id;
    private String nickname;
    private String username;
    private String password;
}
```
```java
public class ApplicationTest {
    @Test
    public void test1() throws Exception {
        ApplicationContext context = new ClassPathXmlApplicationContext("src/applicationContext.xml");
        DruidDataSource dataSource = context.getBean("dataSource", DruidDataSource.class);
        DruidPooledConnection connection = dataSource.getConnection();
        PreparedStatement preparedStatement = connection.prepareStatement("select * from tb_user");
        ResultSet resultSet = preparedStatement.executeQuery();
        // User列表
        List<User> list = new ArrayList<>();
        while(resultSet.next()){
            User user = new User(
                    resultSet.getInt("id"),
                    resultSet.getString("nickname"),
                    resultSet.getString("username"),
                    resultSet.getString("password"));
            list.add(user);
        }
        list.forEach(System.out::println);
    }
}
```
```properties
三月 11, 2023 10:56:36 下午 com.alibaba.druid.pool.DruidAbstractDataSource warn
警告: oracle.jdbc.driver.OracleDriver is deprecated.Having use oracle.jdbc.OracleDriver.
三月 11, 2023 10:56:36 下午 com.alibaba.druid.pool.DruidDataSource info
信息: {dataSource-1} inited
User(id=1, nickname=哈哈, username=admin, password=123456)
User(id=2, nickname=呵呵, username=abcd, password=456)
User(id=3, nickname=嘿嘿, username=aan, password=1234)
Process finished with exit code 0
```
![image-20230311230401198](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303112304328.png)
[项目](https://www.123pan.com/s/tMU0Vv-K2iUd.html)、[lib包](https://www.123pan.com/s/tMU0Vv-u2iUd.html)
