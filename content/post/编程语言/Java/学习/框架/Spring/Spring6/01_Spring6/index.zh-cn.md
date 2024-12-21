---
title: 01_Spring6
description: 01_Spring6
date: 2023-03-11
slug: 01_Spring6
image: 202412212133331.png
categories:
    - Spring
---

# Spring6学习01
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.example</groupId>
        <artifactId>Spring6</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <groupId>com.example</groupId>
    <artifactId>Spring001</artifactId>
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <!--spring ioc-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>6.0.2</version>
        </dependency>
        <!--junit5测试-->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.3.1</version>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
        <!--log4j2的依赖-->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.19.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j2-impl</artifactId>
            <version>2.19.0</version>
        </dependency>
    </dependencies>
</project>
```
### xml配置
> beans.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User"/>
</beans>
```
>log4j2.xml，配置log4j2的日志输出
>
>其中log和RollingFile会将日志输出到文件中，如果文件夹不存在，会自动创建
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <loggers>
        <!--
            level指定日志级别，从低到高的优先级：
                TRACE < DEBUG < INFO < WARN < ERROR < FATAL
                trace：追踪，是最低的日志级别，相当于追踪程序的执行
                debug：调试，一般在开发中，都将其设置为最低的日志级别
                info：信息，输出重要的信息，使用较多
                warn：警告，输出警告的信息
                error：错误，输出错误信息
                fatal：严重错误
        -->
        <root level="DEBUG">
            <appender-ref ref="spring6log"/>
            <appender-ref ref="RollingFile"/>
            <appender-ref ref="log"/>
        </root>
    </loggers>
    <appenders>
        <!--输出日志信息到控制台-->
        <console name="spring6log" target="SYSTEM_OUT">
            <!--控制日志输出的格式-->
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss SSS} [%t] %-3level %logger{1024} - %msg%n"/>
        </console>
        <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，适合临时测试用-->
        <File name="log" fileName="D:/spring6_log/test.log" append="false">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>
        </File>
        <!-- 这个会打印出所有的信息，
            每次大小超过size，
            则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，
            作为存档-->
        <RollingFile name="RollingFile" fileName="D:/spring6_log/app.log"
                     filePattern="log/$${date:yyyy-MM}/app-%d{MM-dd-yyyy}-%i.log.gz">
            <PatternLayout pattern="%d{yyyy-MM-dd 'at' HH:mm:ss z} %-5level %class{36} %L %M - %msg%xEx%n"/>
            <SizeBasedTriggeringPolicy size="50MB"/>
            <!-- DefaultRolloverStrategy属性如不设置，
            则默认为最多同一文件夹下7个文件，这里设置了20 -->
            <DefaultRolloverStrategy max="20"/>
        </RollingFile>
    </appenders>
</configuration>
```
## 获取bean
> 1. 根据id获取bean
> 2. 根据类型获取bean
> 3. 根据id和类型获取bean
```java
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
    private final Logger logger= LoggerFactory.getLogger(ApplicationTest.class);
    @Test
    public void test1(){
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        // 1.根据id获取bean
        User user1 = (User) context.getBean("user");
        // 2.根据类型获取bean
        User user2 = context.getBean(User.class);
        // 3.根据id和类型获取bean
        User user3 = context.getBean("user", User.class);
        // 日志输出
        logger.info("user1:{}",user1);
        logger.info("user2:{}",user2);
        logger.info("user3:{}",user3);
    }
}
```
> `需要注意的是`如果根据类型获取，要求IOC中指定类型的bean只能有一个
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User"/>
    <bean id="user1" class="com.example.domain.User"/>
</beans>
```
```java
public class ApplicationTest {
    private final Logger logger= LoggerFactory.getLogger(ApplicationTest.class);
    @Test
    public void test1(){
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        // 根据类型获取bean
        User user2 = context.getBean(User.class);
        // 日志输出
        logger.info("user2:{}",user2);
    }
}
```
> `报错信息如下`
```error
org.springframework.beans.factory.NoUniqueBeanDefinitionException:
No qualifying bean of type 'com.example.domain.User' available: 
expected single matching bean but found 2: user,user1
```
> UserDao userDao=new UserDaoImpl();
>
> bean配置能否实现如上效果？如果有多个Impl的实现类呢？
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="userDaoImpl" class="com.example.dao.impl.UserDaoImpl"/>
</beans>
```
```java
public interface UserDao {
    void testDao();
}
```
```java
public class UserDaoImpl implements UserDao {
    @Override
    public void testDao() {
        System.out.println("这里是userDaoImpl");
    }
}
```
```java
public class ApplicationTest {
    private final Logger logger= LoggerFactory.getLogger(ApplicationTest.class);
    @Test
    public void test1(){
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        UserDao userDao = context.getBean(UserDao.class);
        logger.info("userDao:{}",userDao);
        userDao.testDao();
    }
}
```
> 可以通过类型直接获取到
>
> 但是当新建第二个impl实现类时，无法根据类型获取到，报错和之前一样
> 可以根据bean的名称获取到、也可以根据bean的名称+类型获取到
>
> **结论**
>
> 根据类型来获取bean时，在满足bean唯一性的前提下，其实只是看：『对象 **instanceof** 指定的类型』的返回结果，只要返回的是true就可以认定为和类型匹配，能够获取到。
>
> java中，instanceof运算符用于判断前面的对象是否是后面的类，或其子类、实现类的实例。如果是返回true，否则返回false。也就是说：用instanceof关键字做判断时， instanceof 操作符的左右操作必须有继承或实现关系
```java
public class ApplicationTest {
    private final Logger logger= LoggerFactory.getLogger(ApplicationTest.class);
    @Test
    public void test1(){
        UserDaoImpl userDaoImpl = new UserDaoImpl();
        System.out.println(userDaoImpl instanceof UserDao);
    }
}
// 结果为 true
```
## 依赖注入-setter注入&&构造器注入
```java
public class ApplicationTest {
    private final Logger logger= LoggerFactory.getLogger(ApplicationTest.class);
    @Test
    public void test1(){
        // set方法
        User user1 = new User();
        user1.setUid(1);
        user1.setName("哈哈");
        user1.setGender("男");
        user1.setAge(15);
        // 构造器
        User user2 = new User(1, "哈哈", "男", 15);
        System.out.println("user1:"+user1);
        System.out.println("user2:"+user2);
    }
}
```
```
user1:User(uid=1, name=哈哈, gender=男, age=15)
user2:User(uid=1, name=哈哈, gender=男, age=15)
```
> 这里想测试一下这两个user对象是否相等
>
> 预想的情况应该是 false，false
>
> 因为Object里的equals方法就是 ==
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303111543951.png" alt="image-20230311154259756" style="zoom:80%;" />
```java
System.out.println("user1 && user2是否相等:"+user1.equals(user2));
System.out.println("user1 && user2是否相等:"+(user1==user2));
```
> 但答案是 true，false，我猜是@Data的问题
>
> 我一直以为@Data=get+set+toString
>
> 但其实@Data =get、set、equals、hashCode、toString
### setter注入
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <property name="uid" value="1"/>
        <property name="name" value="马小跳"/>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
    </bean>
    <bean id="user1" class="com.example.domain.User">
        <property name="uid" value="2"/>
        <property name="name" value="马跳"/>
        <property name="gender" value="女"/>
        <property name="age" value="22"/>
    </bean>
</beans>
```
### 构造器注入
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <constructor-arg name="uid" value="1"/>
        <constructor-arg name="name" value="马小跳"/>
        <constructor-arg name="gender" value="男"/>
        <constructor-arg name="age" value="15"/>
    </bean>
</beans>
```
> 前提
> 使用setter注入，需要提前生成set方法
> 使用构造器注入，需要提前生成全参数的构造方法
### 特殊值处理
#### null
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <constructor-arg name="uid" value="1"/>
        <constructor-arg name="name" value=""/>
        <constructor-arg name="gender" value="男"/>
        <constructor-arg name="age" value="15"/>
    </bean>
</beans>
```
```java
public class ApplicationTest {
    private final Logger logger= LoggerFactory.getLogger(ApplicationTest.class);
    @Test
    public void test1(){
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        User bean = context.getBean(User.class);
        System.out.println(bean.getName() == null);
        System.out.println(Objects.equals(bean.getName(), ""));
    }
}
```
> 结果是 false，true
>
> 此时name的值并不是null，是空字符串，如果希望它的值就是null，那么xml应该如下配置`（set同理）`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <constructor-arg name="uid" value="1"/>
        <constructor-arg name="name">
            <null/>
        </constructor-arg>
        <constructor-arg name="gender" value="男"/>
        <constructor-arg name="age" value="15"/>
    </bean>
</beans>
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <property name="uid" value="1"/>
        <property name="name">
            <null/>
        </property>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
    </bean>
</beans>
```
#### 特殊符号
> 写法1
```xml
<!-- 小于号在XML文档中用来定义标签的开始，不能随便使用 -->
<!-- 解决方案一：使用XML实体来代替 -->
<property name="expression" value="a &lt; b"/>
```
> 写法2
```xml
<property name="expression">
    <!-- 解决方案二：使用CDATA节 -->
    <!-- CDATA中的C代表Character，是文本、字符的含义，CDATA就表示纯文本数据 -->
    <!-- XML解析器看到CDATA节就知道这里是纯文本，就不会当作XML标签或属性来解析 -->
    <!-- 所以CDATA节中写什么符号都随意 -->
    <value><![CDATA[a < b]]></value>
</property>
```
#### 实体
> 方法1 外部引入
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Phone {
    private Integer pid;
    private String name;
    private Float price;
}
```
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer uid;
    private String name;
    private String gender;
    private Integer age;
    // 包含了phone类
    private Phone phone;
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <property name="uid" value="1"/>
        <property name="name" value="马小跳"/>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
        <!-- ref 关联 phone -->
        <property name="phone" ref="phone"/>
    </bean>
    <bean id="phone" class="com.example.domain.Phone">
        <property name="pid" value="001"/>
        <property name="name" value="不锈钢手机"/>
        <property name="price" value="4999.9"/>
    </bean>
</beans>
```
![image-20230311160628197](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303111606408.png)
```java
public class ApplicationTest {
    private final Logger logger= LoggerFactory.getLogger(ApplicationTest.class);
    @Test
    public void test1(){
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        User bean = context.getBean(User.class);
        System.out.println(bean);
    }
}
```
> 方法2 内部bean
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <property name="uid" value="1"/>
        <property name="name" value="马小跳"/>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
        <!-- 不填写value或ref，将bean写在property内 -->
        <property name="phone">
            <bean class="com.example.domain.Phone">
                <property name="pid" value="001"/>
                <property name="name" value="不锈钢手机"/>
                <property name="price" value="4999.9"/>
            </bean>
        </property>
    </bean>
</beans>
```
> 方法3 级联属性赋值
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <property name="uid" value="1"/>
        <property name="name" value="马小跳"/>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
        <property name="phone" ref="phone"/>
        <!-- 会将price的值进行覆盖 -->
        <property name="phone.price" value="3999.9"/>
    </bean>
    <bean id="phone" class="com.example.domain.Phone">
        <property name="pid" value="001"/>
        <property name="name" value="不锈钢手机"/>
        <property name="price" value="4999.9"/>
    </bean>
</beans>
```
> 总结：
>
> 以上都是set注入的写法，构造器注入的写法是一样的
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <constructor-arg name="uid" value="1"/>
        <constructor-arg name="name" value="马小跳"/>
        <constructor-arg name="gender" value="男"/>
        <constructor-arg name="age" value="15"/>
        <constructor-arg name="phone" ref="phone"/>
    </bean>
    <bean id="phone" class="com.example.domain.Phone">
        <constructor-arg name="pid" value="001"/>
        <constructor-arg name="name" value="马小跳"/>
        <constructor-arg name="price" value="4999.9"/>
    </bean>
</beans>
```
#### 数组
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer uid;
    private String name;
    private String gender;
    private Integer age;
    private String[] hobby;
    private Phone phone;
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <property name="uid" value="1"/>
        <property name="name" value="马小跳"/>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
        <property name="hobby">
            <array>
                <value>唱</value>
                <value>跳</value>
            </array>
        </property>
        <property name="phone" ref="phone"/>
    </bean>
    <bean id="phone" class="com.example.domain.Phone">
        <property name="pid" value="001"/>
        <property name="name" value="不锈钢手机"/>
        <property name="price" value="4999.9"/>
    </bean>
</beans>
```
#### 集合
##### List
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer uid;
    private String name;
    private String gender;
    private Integer age;
    // 实体对象集合
    private List<Phone> phones;
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <property name="uid" value="1"/>
        <property name="name" value="马小跳"/>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
        <property name="phones">
            <list>
                <ref bean="phone"/>
                <ref bean="phone1"/>
            </list>
        </property>
    </bean>
    <bean id="phone" class="com.example.domain.Phone">
        <property name="pid" value="001"/>
        <property name="name" value="不锈钢手机"/>
        <property name="price" value="4999.9"/>
    </bean>
    <bean id="phone1" class="com.example.domain.Phone">
        <property name="pid" value="002"/>
        <property name="name" value="塑料手机"/>
        <property name="price" value="3999.9"/>
    </bean>
</beans>
```
> 上面演示的是 实体对象集合，如果是字符串集合，可使用value，而不是ref
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer uid;
    private String name;
    private String gender;
    private Integer age;
    // 字符串集合
    private List<String> phones;
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <property name="uid" value="1"/>
        <property name="name" value="马小跳"/>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
        <property name="phones">
            <list>
                <value>呵呵</value>
                <value>哈哈</value>
            </list>
        </property>
    </bean>
    <bean id="phone" class="com.example.domain.Phone">
        <property name="pid" value="001"/>
        <property name="name" value="不锈钢手机"/>
        <property name="price" value="4999.9"/>
    </bean>
    <bean id="phone1" class="com.example.domain.Phone">
        <property name="pid" value="002"/>
        <property name="name" value="塑料手机"/>
        <property name="price" value="3999.9"/>
    </bean>
</beans>
```
##### Map
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer uid;
    private String name;
    private String gender;
    private Integer age;
    private Map<String,Phone> phoneMap;
}
```
> 写法1
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <property name="uid" value="1"/>
        <property name="name" value="马小跳"/>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
        <property name="phoneMap">
            <map>
                <entry>
                    <key>
                        <value>哈哈</value>
                    </key>
                    <ref bean="phone"/>
                </entry>
            </map>
        </property>
    </bean>
    <bean id="phone" class="com.example.domain.Phone">
        <property name="pid" value="001"/>
        <property name="name" value="不锈钢手机"/>
        <property name="price" value="4999.9"/>
    </bean>
</beans>
```
> 写法2
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <property name="uid" value="1"/>
        <property name="name" value="马小跳"/>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
        <property name="phoneMap">
            <map>
                <entry key="第一部手机" value-ref="phone"/>
            </map>
        </property>
    </bean>
    <bean id="phone" class="com.example.domain.Phone">
        <property name="pid" value="001"/>
        <property name="name" value="不锈钢手机"/>
        <property name="price" value="4999.9"/>
    </bean>
</beans>
```
> 以上的map集合里只有一个值，多个值的写法一样
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <property name="uid" value="1"/>
        <property name="name" value="马小跳"/>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
        <property name="phoneMap">
            <map>
                <entry key="第一部手机" value-ref="phone"/>
                <entry key="第二部手机" value-ref="phone1"/>
            </map>
        </property>
    </bean>
    <bean id="phone" class="com.example.domain.Phone">
        <property name="pid" value="001"/>
        <property name="name" value="不锈钢手机"/>
        <property name="price" value="4999.9"/>
    </bean>
    <bean id="phone1" class="com.example.domain.Phone">
        <property name="pid" value="002"/>
        <property name="name" value="塑料手机"/>
        <property name="price" value="3999.9"/>
    </bean>
</beans>
```
##### 引用集合类型的bean   util命令空间&&p命名空间
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer uid;
    private String name;
    private String gender;
    private Integer age;
    private List<Lesson> lessons;
    private Map<String,Phone> phoneMap;
}
```
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Lesson {
    private Integer lid;
    private String name;
}
```
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Phone {
    private Integer pid;
    private String name;
    private Float price;
}
```
> 使用之前的写法
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="user" class="com.example.domain.User">
        <property name="uid" value="1"/>
        <property name="name" value="马小跳"/>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
        <property name="lessons">
            <list>
                <ref bean="lesson1"/>
                <ref bean="lesson2"/>
            </list>
        </property>
        <property name="phoneMap">
            <map>
                <entry key="第一部手机" value-ref="phone1"/>
                <entry key="第二部手机" value-ref="phone2"/>
            </map>
        </property>
    </bean>
    <!-- lesson1 -->
    <bean id="lesson1" class="com.example.domain.Lesson">
        <property name="lid" value="0001"/>
        <property name="name" value="java开发"/>
    </bean>
    <!-- lesson2 -->
    <bean id="lesson2" class="com.example.domain.Lesson">
        <property name="lid" value="0002"/>
        <property name="name" value="python开发"/>
    </bean>
    <!-- phone1 -->
    <bean id="phone1" class="com.example.domain.Phone">
        <property name="pid" value="001"/>
        <property name="name" value="不锈钢手机"/>
        <property name="price" value="4999.9"/>
    </bean>
    <!-- phone2 -->
    <bean id="phone2" class="com.example.domain.Phone">
        <property name="pid" value="002"/>
        <property name="name" value="塑料手机"/>
        <property name="price" value="3999.9"/>
    </bean>
</beans>
```
> 第二种写法
>
> 这里需要更改xml上面的命名空间
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
    <bean id="user" class="com.example.domain.User">
        <property name="uid" value="1"/>
        <property name="name" value="马小跳"/>
        <property name="gender" value="男"/>
        <property name="age" value="15"/>
        <property name="lessons" ref="lessonList"/>
        <property name="phoneMap" ref="phoneMap"/>
    </bean>
    <util:list id="lessonList">
        <ref bean="lesson1"/>
        <ref bean="lesson2"/>
    </util:list>
    <util:map id="phoneMap">
        <entry key="第一部手机" value-ref="phone1"/>
        <entry key="第二部手机" value-ref="phone2"/>
    </util:map>
    <!-- lesson1 -->
    <bean id="lesson1" class="com.example.domain.Lesson">
        <property name="lid" value="0001"/>
        <property name="name" value="java开发"/>
    </bean>
    <!-- lesson2 -->
    <bean id="lesson2" class="com.example.domain.Lesson">
        <property name="lid" value="0002"/>
        <property name="name" value="python开发"/>
    </bean>
    <!-- phone1 -->
    <bean id="phone1" class="com.example.domain.Phone">
        <property name="pid" value="001"/>
        <property name="name" value="不锈钢手机"/>
        <property name="price" value="4999.9"/>
    </bean>
    <!-- phone2 -->
    <bean id="phone2" class="com.example.domain.Phone">
        <property name="pid" value="002"/>
        <property name="name" value="塑料手机"/>
        <property name="price" value="3999.9"/>
    </bean>
</beans>
```
![image-20230311165134848](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303111651961.png)
> 第三种写法 p命名空间注入
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
    <bean id="user" class="com.example.domain.User"
          p:uid="1" p:name="马小跳" p:gender="男" p:age="15"
          p:lessons-ref="lessonList"
          p:phoneMap-ref="phoneMap"/>
    <util:list id="lessonList">
        <ref bean="lesson1"/>
        <ref bean="lesson2"/>
    </util:list>
    <util:map id="phoneMap">
        <entry key="第一部手机" value-ref="phone1"/>
        <entry key="第二部手机" value-ref="phone2"/>
    </util:map>
    <!-- lesson1 -->
    <bean id="lesson1" class="com.example.domain.Lesson">
        <property name="lid" value="0001"/>
        <property name="name" value="java开发"/>
    </bean>
    <!-- lesson2 -->
    <bean id="lesson2" class="com.example.domain.Lesson">
        <property name="lid" value="0002"/>
        <property name="name" value="python开发"/>
    </bean>
    <!-- phone1 -->
    <bean id="phone1" class="com.example.domain.Phone">
        <property name="pid" value="001"/>
        <property name="name" value="不锈钢手机"/>
        <property name="price" value="4999.9"/>
    </bean>
    <!-- phone2 -->
    <bean id="phone2" class="com.example.domain.Phone">
        <property name="pid" value="002"/>
        <property name="name" value="塑料手机"/>
        <property name="price" value="3999.9"/>
    </bean>
</beans>
```
![image-20230311170408895](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303111704009.png)
