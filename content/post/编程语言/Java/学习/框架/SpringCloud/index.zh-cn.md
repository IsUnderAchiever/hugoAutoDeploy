---
title: SpringCloud
description: SpringCloud
date: 2024-04-10
slug: SpringCloud
image: 202412212133331.png
categories:
    - Spring
---

# SpringCloud
## 服务的注册发现
### Http请求
> 微服务远程调用可以采用`Http`和`RPC`的方式
#### 项目搭建
注册`RestTemplate`，通过`RestTemplate`来实现http请求调用其他服务
> 1. 新建一个空项目，名为`cloud-demo`
> 2. 在`cloud-demo`下新建一个`order`项目和`user`项目(这里采用`jdk17`新建`springboot3`工程)
> cloud-demo/pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>cloud-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>cloud-demo</name>
    <description>cloud-demo</description>
    <packaging>pom</packaging>
    <modules>
        <module>order</module>
        <module>user</module>
    </modules>
    <properties>
        <java.version>17</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>3.0.2</spring-boot.version>
        <spring-cloud-alibaba.version>2022.0.0.0-RC2</spring-cloud-alibaba.version>
        <spring-cloud.version>2022.0.0-RC2</spring-cloud.version>
        <mysql.version>8.3.0</mysql.version>
        <lombok.version>1.18.24</lombok.version>
        <mybatis-plus.version>3.5.5</mybatis-plus.version>
    </properties>
    <dependencyManagement>
        <dependencies>
			<dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-boot-starter</artifactId>
                <version>${mybatis-plus.version}</version>
            </dependency>
            <dependency>
                <groupId>com.mysql</groupId>
                <artifactId>mysql-connector-j</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```
> user/pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>cloud-demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <artifactId>user</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>user</name>
    <description>user</description>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```
> order与user一样
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>cloud-demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <artifactId>order</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>order</name>
    <description>order</description>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```
> user/application.properties
```properties
spring.application.name=user-service
# 应用服务 WEB 访问端口
server.port=8080
# 数据库驱动：
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 数据库连接地址
spring.datasource.url=jdbc:mysql://localhost:3306/cloud-user?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
# 数据库用户名&密码：
spring.datasource.username=root
spring.datasource.password=123456
# 数据源名称
spring.datasource.name=defaultDataSource
# 配置mybatis-plus 打印sql日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
# xml文件路径
mybatis-plus.mapper-locations=classpath:/mapper/**/*.xml
# 配置mybatis-plus 包路径
mybatis-plus.type-aliases-package=com.example.user.domain
# 配置全局默认主键类型，实体类不用加@TableId(value ="id",type = IdType.AUTO)
mybatis-plus.global-config.db-config.id-type=auto
```
> order/application.properties
```properties
spring.application.name=order-service
# 应用服务 WEB 访问端口
server.port=8090
# 数据库驱动：
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 数据库连接地址
spring.datasource.url=jdbc:mysql://localhost:3306/cloud-order?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
# 数据库用户名&密码：
spring.datasource.username=root
spring.datasource.password=123456
# 数据源名称
spring.datasource.name=defaultDataSource
# 配置mybatis-plus 打印sql日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
# xml文件路径
mybatis-plus.mapper-locations=classpath:/mapper/**/*.xml
# 配置mybatis-plus 包路径
mybatis-plus.type-aliases-package=com.example.order.domain
# 配置全局默认主键类型，实体类不用加@TableId(value ="id",type = IdType.AUTO)
mybatis-plus.global-config.db-config.id-type=auto
```
> 这里使用`idea`自带的数据库管理工具来新建数据库
![image-20240404085254211](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404085254211.png)
> user数据库如下，输入`cloud-user`，order同理`cloud-order`
>
> 右键数据库，选择创建表
![image-20240404085443159](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404085443159.png)
![image-20240404085814092](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404085814092.png)
```sql
create table user
(
    id       int auto_increment comment '用户编号',
    username varchar(20) null comment '账号',
    address  varchar(50) null comment '地址',
    constraint user_pk
        primary key (id)
)
    comment '用户表';
    
INSERT INTO `cloud-user`.user (id, username, address) VALUES (1, 'admin', '湖北');
INSERT INTO `cloud-user`.user (id, username, address) VALUES (2, 'zhangsan', '湖南');
INSERT INTO `cloud-user`.user (id, username, address) VALUES (3, 'lisi', '山西');
```
```sql
create table `orders`
(
    id      int auto_increment comment '订单编号',
    name    varchar(20) null comment '名称',
    price   decimal     null comment '价格',
    num     int         null comment '数量',
    user_id int         null comment '用户编号',
    constraint order_pk
        primary key (id)
)
    comment '订单表';
    
INSERT INTO `cloud-order`.`orders` (id, name, price, num, user_id) VALUES (1, '小米手机', 4999, 1, 1);
INSERT INTO `cloud-order`.`orders` (id, name, price, num, user_id) VALUES (2, '华为手机', 5999, 3, 1);
INSERT INTO `cloud-order`.`orders` (id, name, price, num, user_id) VALUES (3, '魅族手机', 3999, 2, 2);
INSERT INTO `cloud-order`.`orders` (id, name, price, num, user_id) VALUES (4, 'OPPO手机', 4000, 1, 3);
```
> 我们的`需求`是:**`根据订单id查询订单的同时，把订单所属的用户信息一起返回`**
>
> 这里就是`MyBatisX`一键生成`mapper`、`service`代码了
>
> 这里需要注意个问题`表名`不要叫`order`，order属于数据库的关键字，选择`orders`
![image-20240404090905809](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404090905809.png)
#### 编写controller代码
```java
@RestController
@RequestMapping("order")
public class OrdersController {
    @Autowired
    private OrdersService ordersService;
    @PostMapping
    public Boolean save(@RequestBody Orders orders) {
        return ordersService.saveOrUpdate(orders);
    }
    @DeleteMapping("/{id}")
    public Boolean delete(@PathVariable Integer id) {
        return ordersService.removeById(id);
    }
    @GetMapping
    public List<Orders> findAll() {
        return ordersService.list();
    }
    @GetMapping("/{id}")
    public Orders findOne(@PathVariable Integer id) {
        return ordersService.getById(id);
    }
    @GetMapping("/page")
    public Page<Orders> findPage(@RequestParam Integer pageNum,
                                @RequestParam Integer pageSize) {
        return ordersService.page(new Page<>(pageNum, pageSize));
    }
}
```
```java
@RestController
@RequestMapping("user")
public class UserController {
    @Autowired
    private UserService userService;
    @PostMapping
    public Boolean save(@RequestBody User user) {
        return userService.saveOrUpdate(user);
    }
    @DeleteMapping("/{id}")
    public Boolean delete(@PathVariable Integer id) {
        return userService.removeById(id);
    }
    @GetMapping
    public List<User> findAll() {
        return userService.list();
    }
    @GetMapping("/{id}")
    public User findOne(@PathVariable Integer id) {
        return userService.getById(id);
    }
    @GetMapping("/page")
    public Page<User> findPage(@RequestParam Integer pageNum,
                                @RequestParam Integer pageSize) {
        return userService.page(new Page<>(pageNum, pageSize));
    }
}
```
> 输入http://localhost:8080/user/1即可访问到`user`的结果
>
> 输入http://localhost:8090/order/1即可访问到`order`的结果
>
> **`根据订单id查询订单的同时，把订单所属的用户信息一起返回`** 需求可以转变为 在`OrderController`中请求一下user的结果，然后封装返回
**配置Order服务的RestTemplate**
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;
/**
 * Http请求调用配置
 * @author: 不是菜狗爱编程
 * @date: 2024/04/04/8:40
 * @description:
 */
@Configuration
public class HttpConfig {
    /**
     * 创建RestTemplate并注入spring容器
     *
     * @return {@link RestTemplate}
     */
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```
> 复制`User`实体类到`Order`服务
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User implements Serializable {
    private Integer id;
    private String username;
    private String address;
}
```
> `Order`服务新增一个`User`属性
```java
@TableName(value ="orders")
@Data
public class Orders implements Serializable {
    @TableId(type = IdType.AUTO)
    private Integer id;
    private String name;
    private Integer price;
    private Integer num;
    private Integer userId;
    @TableField(exist = false)
    private User user;
    @TableField(exist = false)
    private static final long serialVersionUID = 1L;
}
```
> 修改`OrdersController`方法
```java
@RestController
@RequestMapping("order")
public class OrdersController {
    private static final String USER_SERVICE_URL="http://localhost:8080/user/";
    @Autowired
    private OrdersService ordersService;
    @Autowired
    private RestTemplate restTemplate;
    @GetMapping("/{id}")
    public Orders findOne(@PathVariable Integer id) {
        Orders orders = ordersService.getById(id);
        User user = restTemplate.getForObject(USER_SERVICE_URL + orders.getUserId(), User.class);
       orders.setUser(user);
        return orders;
    }
}
```
> 访问http://localhost:8090/order/1
![image-20240404112131695](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404112131695.png)
> 以上，实现了跨入服务的远程调用
### Eureka
#### 概念
服务提供者: 一次业务中，被其它微服务调用的服务。(提供接口给其它微服务)
服务消费者: 一次业务中，调用其它微服务的服务。(调用其它微服务提供的接口)
以上代码的实现中，User是服务的提供者，Order是服务的消费者
代码中使用`硬编码`将url写在代码中，将来如果部署成集群，则`硬编码`应该怎么办？
应该选择哪一台？选的那一台实例是否依然正常？
`Eureka`、`Nacos`可以实现上述需求
![image-20240404112656015](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404112656015.png)
> 在每一个服务启动时，将自己的服务信息注册到`Eureka`中，这时候`Eureka`中已经存储了相关的`Order`和`User`服务的信息
>
> 在`Order`服务调用`User`服务时，`Order`服务前往`Eureka`中找到`User`服务的url
>
> 那怎么知道找到的`User`服务是否依然正常运行？
>
> 服务会每30s向`Eureka`发送心跳，如果心跳停止，说明服务异常
在`Eureka`架构中，微服务角色有两类:
1. EurekaServer: 服务端，注册中心
   - 记录服务信息
   - 心跳监控
2. EurekaClient: 客户端
   1. Provider: 服务提供者，例如案例中的user-service
      - 注册自己的信息到EurekaServer
      - 每隔30秒向EurekaServer发送心跳
   2. consumer:服务消费者，例如案例中的order-service
      - 根据服务名称从EurekaServer拉取服务列表
      - 基于服务列表做负载均衡，选中一个微服务后发起远程调用
#### 搭建Eureka服务
1. 引入依赖
2. 编写启动类，添加`@EnableEurekaServer`注解
3. 配置`application.properties`
> cloud-demo/pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>cloud-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>cloud-demo</name>
    <description>cloud-demo</description>
    <packaging>pom</packaging>
    <modules>
        <module>order</module>
        <module>user</module>
        <module>eureka</module>
    </modules>
    <properties>
        <java.version>17</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>3.0.2</spring-boot.version>
        <spring-cloud.version>2022.0.0-RC2</spring-cloud.version>
        <mysql.version>8.3.0</mysql.version>
        <lombok.version>1.18.24</lombok.version>
        <mybatis-plus.version>3.5.5</mybatis-plus.version>
    </properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-boot-starter</artifactId>
                <version>${mybatis-plus.version}</version>
            </dependency>
            <dependency>
                <groupId>com.mysql</groupId>
                <artifactId>mysql-connector-j</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```
> eureka/pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>cloud-demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <artifactId>eureka</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eureka</name>
    <description>eureka</description>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```
```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```
```properties
# 应用服务 WEB 访问端口
server.port=8070
# 是否将自己注册到注册中心
eureka.client.register-with-eureka=false
# 是否从EurekaServer抓取已有的注册信息，默认为true。 单节点无所谓， 集群必须设置为true才能配合ribbon使用负载均衡
eureka.client.fetch-registry=false
eureka.client.service-url.defaultZone=http://localhost:${server.port}/eureka/
# 关闭自我保护机制
eureka.server.enable-self-preservation=false
eureka.server.eviction-interval-timer-in-ms=2000
```
![image-20240404125646354](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404125646354.png)
`Eureka`配置成功，只是此时并没有服务注册到`Eureka`
#### 服务注册
> 配置`user`和`order`，将服务注册进`eureka`
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```
```properties
eureka.instance.hostname=${spring.application.name}
# 是否显示ip
eureka.instance.prefer-ip-address=true
#Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
eureka.instance.lease-renewal-interval-in-seconds=3
#Eureka,服务端在收到最后-次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
eureka.instance.lease-expiration-duration-in-seconds=5
eureka.client.service-url.defaultZone=http://localhost:8070/eureka/
```
![image-20240404130043753](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404130043753.png)
> 可以看到此时服务已经注册到`Eureka`了，但这些服务都是一个实例，启动多个实例试试
![image-20240404130440728](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404130440728.png)
> 如果找不到`Services`，可以在`View - Tool Windows`处添加
![image-20240404130540152](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404130540152.png)
> 右键服务，选择`Copy Configuration`
![image-20240404130611273](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404130611273.png)
![image-20240404130728336](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404130728336.png)
![image-20240404130811273](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404130811273.png)
```sh
# 配置服务启动端口
-Dserver.port=8081
```
> 右键启动该实例
![image-20240404130916547](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404130916547.png)
> 前往`Eureka`页面查看
![image-20240404130945137](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404130945137.png)
#### 服务发现
> 我们希望`Order`服务调用`User`服务时，前往`Eureka`注册中心获取到`User`服务的信息
>
> 而不是使用硬编码的形式写死`User`服务的`ip`和`端口`
1. 修改OrderService的代码，修改访问的url路径，用服务名代替ip、端口:
```java
String url = "http://user-service/user/" + order.getUserId();
```
2. 在order-service项目的启动类OrderApplication中的RestTemplate添加负载均衡注解:
```java
@Bean
@LoadBalanced
public RestTempLate restTempLate() {
    return new RestTemplate() ;
}
```
3. 在启动类山添加`@EnableDiscoveryClient`注解
> 修改`HttpConfig`的代码
```java
@Configuration
public class HttpConfig {
    /**
     * 创建RestTemplate并注入spring容器
     *
     * @return {@link RestTemplate}
     */
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```
> 修改`OrdersController`代码
```java
@RestController
@RequestMapping("order")
public class OrdersController {
    @Autowired
    private OrdersService ordersService;
    @Autowired
    private RestTemplate restTemplate;
    
    @GetMapping("/{id}")
    public Orders findOne(@PathVariable Integer id) {
        Orders orders = ordersService.getById(id);
        User user = restTemplate.getForObject("http://user-service/user/" + orders.getUserId(), User.class);
        orders.setUser(user);
        return orders;
    }
}
```
> 如果报错`No instances available for user-service`
>
> 请查看[博客](https://blog.csdn.net/Kevinnsm/article/details/117117061)
> 如果依然不能解决，请查看`application.properties`里是否配置了如下内容，如果是，删除后重启即可解决
```properties
# 是否从EurekaServer抓取已有的注册信息，默认为true。 单节点无所谓， 集群必须设置为true才能配合ribbon使用负载均衡
eureka.client.fetch-registry=false
```
![image-20240404152215166](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404152215166.png)
> 这样就可以通过`服务名`的方式来完成服务之间的调用，哪怕服务启动了`多个实例`
#### 负载均衡
##### 基础
> `Ribbon`实现负载均衡
![image-20240404152632326](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404152632326.png)
> `Ribbon`是如何实现负载均衡的呢？
> 添加了`@LoadBalanced`注解后，表明该`RestTemplate`发起的请求，需要被`Ribbon`来拦截处理
>
> 而拦截的动作是`LoadBalancerInterceptor`来完成的
>
> `idea`中按两下`shift`后搜索即可
![image-20240404153010079](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404153010079.png)
> 而`LoadBalancerInterceptor`类实现了`ClientHttpRequestInterceptor`接口
![image-20240404153426292](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404153426292.png)
> 它会去拦截由客户端发起的`Http`请求，而`RestTemplate`正是一个发`Http`请求的客户端
>
> `LoadBalancerInterceptor`类实现了`intercept`方法
>
> 在`LoadBalancerInterceptor`打断点调试一下，`debug`运行`Order`
>
> 发现请求确实被`LoadBalancerInterceptor`拦截了
![image-20240404153840301](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404153840301.png)
> 可以看到`Order`服务发起了一个`Http`请求，Url是`http://user-service/user/1`
![image-20240404154021714](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404154021714.png)
> 执行完`String serviceName = originalUri.getHost();`代码之后，获取到了主机名`user-service`
![image-20240404154138579](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404154138579.png)
![image-20240404154304632](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404154304632.png)
> 进入`loadBalancer.execute`方法，继续执行
![image-20240404155350933](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404155350933.png)
> 发现这里已经获取到`User`服务的`ip`了，我们运行了多个`User`服务的实例，这里大家可能看到的是`8080`或者`8090`
>
> 是在这一行代码上获取到了`User`服务
![image-20240404155904705](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404155904705.png)
> 那它怎么知道或者按什么规则来获取的`其中一个User服务`呢？
>
> 继续进去，发现可能是在`loadBalancer.choose`获取到的
![image-20240404160035088](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404160035088.png)
> 再往里走就是一个接口了`ReactiveLoadBalancer`
>
> 按住`Ctrl H`查看这个接口的实现类
![image-20240404160119331](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404160119331.png)
![image-20240404160330743](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404160330743.png)
> 这里`Eureka`默认的调度机制其实是`轮询`
![image-20240404160846349](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404160846349.png)
![image-20240404161057944](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404161057944.png)
##### 策略
通过定义IRule实现可以修改负载均衡规则，有两种方式:
代码方式:在配置类中，定义一个新的IRule:(全局配置，不管调用哪个服务，都会采用这个负载均衡规则)
```java
@Bean
pubtic IRuLe randomRuLe() {
	return new RandomRuLe();
}
```
第二种配置方案是`properties`，而这种方法是针对每个服务来配置负载均衡规则
```properties
#负载均衡规则
user-service.ribbon.NFLoadBalancerRuLeCLassName=com.netflix.Loadbalancer.RandomRuLe
```
#### 饥饿加载
Ribbon默认是采用懒加载，即第一 次访问时才会去创建LoadBalanceClient, 请求时间会很长。
而饥饿加载则会在项目启动时创建，降低第一次访问的耗时，通过下面配置开启饥饿加载
```properties
# 开启饥饿加载
ribbon.eager-load.enabled=true
# 对指定服务饥饿加载
ribbon.eager-load.clients=user-service
```
### Nacos
#### 安装
[SpringBoot、SpringCloud、Alibaba版本对应关系](https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E)
[Nacos下载](https://github.com/alibaba/nacos/releases)
选择`2.2.1`版本的`Naocs`
`nacos-2.2.1/bin/startup.cmd`用vscode或者记事本打开
![image-20240404230331406.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404230331406.png)
> 双击发现闪退，在结尾处加上如下代码
```
pause
endlocal
```
![image-20240404230832876](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404230832876.png)
> 再次双击，查看到报错信息，找到[博客](https://blog.csdn.net/u013833472/article/details/129913944)
>
> 还需要配置conf`目录下的`application.properties
>
> **nacos.core.auth.default.token.secret.key**
> 这个参数必须配置为`base64`编码而且不能小于32位，随便找个在线网站编码一下即可
重新启动nacos
![image-20240404231527819](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404231527819.png)
![image-20240404231540377](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404231540377.png)
> 账号密码都是`nacos`
> 在父工程中配置pom.xml
```xml
    <properties>
        <java.version>17</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>3.0.2</spring-boot.version>
        <spring-cloud.version>2022.0.0</spring-cloud.version>
        <spring-cloud-alibaba.version>2022.0.0.0-RC2</spring-cloud-alibaba.version>
        <spring-cloud-netflix.version>2.2.0.RELEASE</spring-cloud-netflix.version>
        <lombok.version>1.18.24</lombok.version>
        <nacos-client.version>1.4.4</nacos-client.version>
    </properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
                <version>${spring-cloud-netflix.version}</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba.nacos</groupId>
                <artifactId>nacos-client</artifactId>
                <version>${nacos-client.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
> `Order`和`User`服务的pom.xml
```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```
> application.properties
```properties
# 端口
server.port=8090
# 服务名
spring.application.name=order-service
spring.cloud.nacos.discovery.server-addr=localhost:8848
spring.cloud.nacos.config.server-addr=localhost:8848
spring.cloud.nacos.config.import-check.enabled=false
```
![image-20240404212510998](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404212510998.png)
> 测试一下负载均衡是否可用
>
> 此时还没有意识到事情的`严重性`，然后就一直报错了
#### 解决报错
![image-20240404212744756](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240404212744756.png)
> 查看[博客](https://blog.csdn.net/weixin_43887184/article/details/124036205)
>
> 因为Netflix的组件从2020年开始停止维护，因此spring cloud会逐渐弃用他家的组件,Ribbon就在其中
>
> 依然没有解决该错误后，前往官网查看，发现官方给了[例子](https://github.com/nacos-group/nacos-examples/tree/master/nacos-spring-cloud-example/nacos-spring-cloud-config-example)
>
> 但是版本跟我们的有区别
```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>com.alibaba.nacos</groupId>
                    <artifactId>nacos-client</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.alibaba.nacos</groupId>
            <artifactId>nacos-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```
> 注意，我我们在`父pom`里配置了`nacos-client`的版本
>
> 我使用官方例子的版本`1.1.0`和`1.2.0`报错如下
```
java.lang.NoClassDefFoundError: com/alibaba/nacos/client/logging/NacosLogging
```
上网查询资料发现应该是版本不匹配，后打开依赖分析，发现有一堆`依赖都不匹配`
![image-20240405223449119](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240405223449119.png)
> `nacos-client`版本改为`1.4.4`后以上报错解决，但是依然无法实现`RestTemplate`调用服务
>
> 问题也是出在依赖上
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-loadbalancer</artifactId>
        </dependency>
<!--        <dependency>-->
<!--            <groupId>org.springframework.cloud</groupId>-->
<!--            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>-->
<!--        </dependency>-->
```
> 将`spring-cloud-starter-netflix-ribbon`改为`spring-cloud-loadbalancer`，这里无需写版本号
>
> 启动后解决问题
#### 依赖
> 重新给一遍`pom`依赖，这里我使用的版本并不需要使用`bootstrap.properties`文件来配置，`application.properties`足矣
> 网上有说需要导入`spring-cloud-starter-bootstrap`依赖，我们这里并不需要
**cloud-demo/pom.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>cloud-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>cloud-demo</name>
    <description>cloud-demo</description>
    <packaging>pom</packaging>
    <modules>
        <module>order</module>
        <module>user</module>
    </modules>
    <properties>
        <java.version>17</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>3.0.2</spring-boot.version>
        <spring-cloud-alibaba.version>2022.0.0.0-RC2</spring-cloud-alibaba.version>
        <spring-cloud.version>2022.0.0-RC2</spring-cloud.version>
        <mysql.version>8.3.0</mysql.version>
        <lombok.version>1.18.24</lombok.version>
        <mybatis-plus.version>3.5.5</mybatis-plus.version>
        <nacos-client.version>1.4.4</nacos-client.version>
    </properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.alibaba.nacos</groupId>
                <artifactId>nacos-client</artifactId>
                <version>${nacos-client.version}</version>
            </dependency>
            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-boot-starter</artifactId>
                <version>${mybatis-plus.version}</version>
            </dependency>
            <dependency>
                <groupId>com.mysql</groupId>
                <artifactId>mysql-connector-j</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```
**order/pom.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>cloud-demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <artifactId>order</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>order</name>
    <description>order</description>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-loadbalancer</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>com.alibaba.nacos</groupId>
                    <artifactId>nacos-client</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.alibaba.nacos</groupId>
            <artifactId>nacos-client</artifactId>
        </dependency>
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <configuration>
                    <mainClass>com.example.order.OrderApplication</mainClass>
                    <skip>true</skip>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    <repositories>
        <repository>
            <id>netflix-candidates</id>
            <name>Netflix Candidates</name>
            <url>https://artifactory-oss.prod.netflix.net/artifactory/maven-oss-candidates</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
</project>
```
> `user`依赖也是一样
**application.properties**
```properties
# 应用服务 WEB 访问端口
server.port=8090
# 服务名
spring.application.name=order-service
# 数据库驱动：
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 数据库连接地址
spring.datasource.url=jdbc:mysql://localhost:3306/cloud-order?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
# 数据库用户名&密码：
spring.datasource.username=root
spring.datasource.password=123456
# 数据源名称
spring.datasource.name=defaultDataSource
# 配置mybatis-plus 打印sql日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
# xml文件路径
mybatis-plus.mapper-locations=classpath:/mapper/**/*.xml
# 配置mybatis-plus 包路径
mybatis-plus.type-aliases-package=com.example.order.domain
spring.cloud.nacos.discovery.server-addr=localhost:8848
spring.cloud.nacos.config.server-addr=localhost:8848
spring.cloud.nacos.config.import-check.enabled=false
```
[项目Gitee地址](https://gitee.com/tongstyle/cloud-demo)，切换到`nacos`分支即可
测试发现，`负载均衡`也可以使用
#### 服务分级存储模型
##### 入门
![image-20240405225820668](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240405225820668.png)
> 每个服务启动多个实例，将这些实例划分到不同的集群
>
> 服务调用尽可能选择本地集群的服务，跨集群调用延迟较高
>
> 本地集群不可访问时，再去访问其它集群
![image-20240405230059593](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240405230059593.png)
> 点击Nacos服务 `"详情"`
![image-20240406062803062](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406062803062.png)
> `集群`是`default`，即没有集群
![image-20240406062856800](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406062856800.png)
> 配置集群属性
1. 修改`application.properties`，添加如下内容
   ```properties
   # 配置集群名称，也就是机房位置，例如JS(江苏)
   spring.cloud.nacos.discovery.cluster-name=JS
   ```
   
2. 在Nacos控制台看到集群的变化
   ![image-20240406065730248](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406065730248.png)
再起一个`User`服务实例，现在是`3`个`User`服务实例了
![image-20240406065908160](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406065908160.png)
> 将`UserApplication1`和`UserApplication2`设置到`JS`江苏集群，将`UserApplication3`设置到`HN`湖南集群
>
> 启动`UserApplication1`和`UserApplication2`，然后将集群改为`HN`，启动`UserApplication3`即可
![image-20240406070456323](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406070456323.png)
**总结**
Nacos服务分级存储模型
1. 一级是服务，例如userservice
2. 二级是集群，例如杭州或上海
3. 三级是实例，例如杭州机房的某台部署了userservice的服务器
##### 服务集群属性
> 这样配置还不算完，我们需要让`Order`服务调用`User`服务时，优先选择本地集群
![image-20240406071112676](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406071112676.png)
> 这里并没有优先使用`JS`的`User`服务，依然采用的是`轮询`方案
>
> 服务在选择实例时，是由`负载均衡`决定的，这里并没有配置，所以采用的是默认`轮询`
```yaml
# 负载均衡规则
userservice:
  ribbon:
    NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule
```
> 实现以上配置之后即可完成实现优先访问本地集群的服务
>
> 实测之后发现只能在较低版本的Nacos中生效，高版本的Nacos已经取消了对Ribbon的支持
>
> 解决办法查看[博客](https://blog.csdn.net/qq_38050728/article/details/121924621),实测可行
>
> 但是随着Spring Cloud Gateway或者其他更先进的API Gateway的广泛应用，以及服务网格（Service Mesh）架构的发展
>
> 例如采用Istio或阿里云的ASM（Application Service Mesh），这种跨可用区或跨集群的流量调度通常会交由服务网格层面处理，而非仅仅依赖客户端SDK内置的负载均衡机制
>
> 那么可能需要转向服务网格或Spring Cloud Gateway级别的路由规则定义，所以负载均衡应该由网关服务来承担，而非消费服务自身
#### 权重设置
**实际部署中会出现这样的场景:**
服务器设备性能有差异，部分实例所在机器性能较好，另一些较差， 我们希望性能好的机器承担更多的用户请求
Nacos提供了权重配置来控制访问频率,权重越大则访问频率越高
**步骤**
1. 在Nacos控制台可以设置实例的权重值，首先选中实例后面的编辑按钮
2. 将权重设置为0.1,测试可以发现8081被访问到的频率大大降低（权重值位于0~1之间）
![image-20240406090249907](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406090249907.png)
> 那这个时候理论上，`8081`的访问次数应该是`8080`的`1/10`，测试发现确实如此，权重配置已经生效
>
> 权重设置为`0`，则该服务不会被访问到
>
> 这种特性可以用以服务的`版本升级`
>
> 正常情况下是不会重启某个服务来实现版本升级的，此时用户还在访问呢，如何在用户无感知的前提下实现版本升级呢？
>
> 在服务启动多个实例的情况下，如`8081`、`8082`、`8083`
> 可以先将`8081`的实例权重设置为`0`，渐渐的`8081`就不会承担用户请求了，此时用户是无感知的
> 这个时候对`8081`停机之后进行升级，然后重启，权重先设置小一点，如果用户请求没有bug，就可以增大权重实现升级了
#### 环境隔离
> Nacos中服务存储和数据存储的最外层都是一个名为`命名空间(namespace)`的东西，用来做最外层隔离
>
> Namespace > Group > Service(服务) > 集群 > 实例
>
> `Group `用作分组，可以将相关度较高的业务划分为一个组
![image-20240406091659048](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406091659048.png)
> 此时就可以看到`dev`命名空间了，但是里面没有任何服务啊
>
> 需要去代码层面添加`命名空间`配置
![image-20240406091755109](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406091755109.png)
```properties
# 命名空间id
spring.cloud.nacos.discovery.namespace=d2455f6d-ed00-41ba-9915-09021bc0df6c
```
![image-20240406091941001](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406091941001.png)
> 重启`order`服务
>
> 发现在`dev`下可以看到`order`服务，而`user`服务仍然还在`public`下
![image-20240406092054361](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406092054361.png)
![image-20240406092129960](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406092129960.png)
> 测试一下能否调用
>
> **不行**
![image-20240406092150037](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406092150037.png)
> 报错如下
```
java.lang.IllegalStateException: No instances available for userservice
```
> 没有找到可用的实例，因为它们处在不同的命名空间下
#### Nacos注册中心原理
![QQ截图20240406132746](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/QQ%E6%88%AA%E5%9B%BE20240406132746.png)
> 服务的`消费者`并不是每次都去拉去`提供者`的信息，而是会将拉去的服务列表进行缓存
>
> 而一直不拉取服务也不行，因为服务提供者的信息可能变化，所以需要每隔一段时间比如30s拉取一次
>
> 但是`Nacos`和`Eureka`还有一些差别在健康检测方面
>
> `Nacos`会将所有的服务提供者分为`临时实例`和`非临时实例`，所有实例默认都是`临时实例`
![image-20240406133353590](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406133353590.png)
> 临时实例在`Nacos`中的健康检测是心跳检测，`临时实例`每隔一段时间向`Nacos`发送心跳，但频率会比`Eureka`的快一些
>
> 一旦心跳检测出现异常、不跳了，`Nacos`就会将其在服务列表中直接`剔除`
>
> `非临时实例`不需要做心跳检测，而是`Nacos`主动发请求询问`实例`是否正常
>
> `Nacos`不会将`非临时实例`在服务列表中`剔除`，而是会等待其恢复健康
>
> `Nacos`和`Eureka`在消费者方面也有一些区别，`Eureka`会每隔30s拉取一次服务
>
> 但如果在`30s`内，有提供者的实例异常，消费者是不知道的呀，此时就会出问题
>
> `Nacos`会主动将`提供者`变更的消息`推`送给`消费者`
>
> `Eureka`主要是采用`pull`，而`Nacos`是`pull`和`push`结合
```properties
# 是否是临时实例
spring.cloud.nacos.discovery.ephemeral=false
```
![image-20240406134615899](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406134615899.png)
> 停止服务
![image-20240406134705641](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406134705641.png)
Nacos支持服务端主动检测提供者状态:临时实例采用心跳模式，非临时实例采用主动检测模式
1. 临时实例心跳不正常会被剔除，非临时实例则不会被剔除
2. Nacos支持服务列表变更的消息推送模式，服务列表更新更及时
3. Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式; Eureka采用AP方式
Nacos与eureka的共同点
都支持服务注册和服务拉取
都支持服务提供者心跳方式做健康检测
#### 配置管理
> 将一些配置信息放在`Nacos`上，如`数据库`、`MyBatis`，而且这些配置可以`动态刷新`，无需重启项目
>
> 一般写的是开关类型的配置，如赋值`true`就开启某个活动
![image-20240406153222831](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406153222831.png)
> 在 Nacos Spring Cloud 中，`dataId` 的完整格式如下：
```
${prefix}-${spring.profiles.active}.${file-extension}
```
- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 `spring.profiles.active` 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。
> `Nacos`配置管理的依赖我们已经引入
```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
```
![image-20240406161144635](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406161144635.png)
> 这里我就直接把`application.properties`中的`DB配置`和`MyBatis`配置写进来了
**order-service-dev.properties**
```properties
# 数据库驱动：
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 数据库连接地址
spring.datasource.url=jdbc:mysql://localhost:3306/cloud-order?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
# 数据库用户名&密码：
spring.datasource.username=root
spring.datasource.password=123456
# 数据源名称
spring.datasource.name=defaultDataSource
# 配置mybatis-plus 打印sql日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
# xml文件路径
mybatis-plus.mapper-locations=classpath:/mapper/**/*.xml
# 配置mybatis-plus 包路径
mybatis-plus.type-aliases-package=com.example.order.domain
```
**user-service-dev.properties**
```properties
# 数据库驱动：
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 数据库连接地址
spring.datasource.url=jdbc:mysql://localhost:3306/cloud-user?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
# 数据库用户名&密码：
spring.datasource.username=root
spring.datasource.password=123456
# 数据源名称
spring.datasource.name=defaultDataSource
# 配置mybatis-plus 打印sql日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
# xml文件路径
mybatis-plus.mapper-locations=classpath:/mapper/**/*.xml
# 配置mybatis-plus 包路径
mybatis-plus.type-aliases-package=com.example.user.domain
```
**`order/application.properties`**
```properties
# 应用服务 WEB 访问端口
server.port=8090
# 服务名
spring.application.name=order-service
# nacos服务发现
spring.cloud.nacos.discovery.server-addr=localhost:8848
# 配置集群名称，也就是机房位置，例如JS(江苏)
spring.cloud.nacos.discovery.cluster-name=JS
# 命名空间id
spring.cloud.nacos.discovery.namespace=d2455f6d-ed00-41ba-9915-09021bc0df6c
# 是否是临时实例
spring.cloud.nacos.discovery.ephemeral=false
# nacos服务配置
spring.cloud.nacos.config.server-addr=localhost:8848
#spring.cloud.nacos.config.import-check.enabled=false
spring.cloud.nacos.config.namespace=d2455f6d-ed00-41ba-9915-09021bc0df6c
# dataId前缀
spring.cloud.nacos.config.prefix=${spring.application.name}
# 环境 (前缀-环境.后缀 如user-dev.properties)
spring.profiles.active=dev
# dataId后缀
spring.cloud.nacos.config.file-extension=properties
spring.config.import=nacos:${spring.cloud.nacos.config.prefix}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}?refresh=true
```
**`user/application.properties`**
```properties
# 应用服务 WEB 访问端口
server.port=8080
# 服务名
spring.application.name=user-service
# nacos服务发现
spring.cloud.nacos.discovery.server-addr=localhost:8848
# 配置集群名称，也就是机房位置，例如JS(江苏)
spring.cloud.nacos.discovery.cluster-name=JS
# 命名空间id
spring.cloud.nacos.discovery.namespace=d2455f6d-ed00-41ba-9915-09021bc0df6c
# 是否是临时实例
spring.cloud.nacos.discovery.ephemeral=false
# nacos服务配置
spring.cloud.nacos.config.server-addr=localhost:8848
spring.cloud.nacos.config.namespace=d2455f6d-ed00-41ba-9915-09021bc0df6c
# dataId前缀
spring.cloud.nacos.config.prefix=${spring.application.name}
# 环境 (前缀-环境.后缀 如user-dev.properties)
spring.profiles.active=dev
# dataId后缀
spring.cloud.nacos.config.file-extension=properties
spring.config.import=nacos:${spring.cloud.nacos.config.prefix}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}?refresh=true
```
> 注意之前配置过`spring.cloud.nacos.config.import-check.enabled=false`，需要删掉。
> 还有`spring.cloud.nacos.config.namespace`一定要指定，因为我们不是配置在默认的`public`，而是自己新建的`dev`。
> **否则在启动项目时会找不到`DB`配置信息报错**
>
> `spring.cloud.nacos.discovery.ephemeral=false`这个也要加上（因为已经在前面被注册成`非临时实例`了）
> 否则会爆错**Current service DEFAULT_GROUP@@order-service is persistent service, can't register ephemeral instance**
> 意思是“DEFAULT_GROUP@@order-service”这个服务是一个持久化服务，不能注册为临时实例。
#### 配置热更新
Nacos中的配置文件变更后，微服务无需重启就可以感知。不过需要通过下面两种配置实现:
- 方式一:在`@Value`注入的变量所在类上添加注解`@RefreshScope`
```java
@RefreshScope
@RestController
@RequestMapping("order")
public class OrdersController {
    @Autowired
    private OrdersService ordersService;
    @Autowired
    private RestTemplate restTemplate;
    @Value("${author.name}")
    private String name;
    @Value("${author.date}")
    private String date;
    @Value("${author.description}")
    private String description;
    @GetMapping("/{id}")
    public Map<String,Object> findOne(@PathVariable Integer id) {
        Map<String, Object> map = new HashMap<>();
        Orders orders = ordersService.getById(id);
        User user = restTemplate.getForObject("http://user-service/user/" + orders.getUserId(), User.class);
        orders.setUser(user);
        map.put("orders",orders);
        map.put("name",name);
        map.put("date",date);
        map.put("description",description);
        return map;
    }
}
```
> 在`order-service-dev.properties`新增配置
```properties
author.name=不是菜狗爱编程
author.date=2024/04/04/9:13
author.description=暂无描述
```
> 重启项目后，访问配置
![image-20240406162720924](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406162720924.png)
> 修改配置后，`不重启`项目
```properties
author.name=不是菜狗爱编程
author.date=2024/04/05/9:13
author.description=第一次发布
```
![image-20240406162807556](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406162807556.png)
- 方式二:使用@ConfigurationProperties注解
新建`Author.java`
```java
@Data
@Component
@AllArgsConstructor
@NoArgsConstructor
@ConfigurationProperties(prefix = "author")
public class Author {
    private String name;
    private String date;
    private String description;
}
```
```java
@RefreshScope
@RestController
@RequestMapping("order")
public class OrdersController {
    @Autowired
    private OrdersService ordersService;
    @Autowired
    private RestTemplate restTemplate;
    @Autowired
    private Author author;
    @GetMapping("/{id}")
    public Map<String,Object> findOne(@PathVariable Integer id) {
        Map<String, Object> map = new HashMap<>();
        Orders orders = ordersService.getById(id);
        User user = restTemplate.getForObject("http://user-service/user/" + orders.getUserId(), User.class);
        orders.setUser(user);
        map.put("orders",orders);
        map.put("author",author);
        return map;
    }
}
```
> 依然可以实现动态更新配置
![image-20240406163238867](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406163238867.png)
#### 多环境配置共享
![image-20240406182210993](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406182210993.png)
> `nacos`/order-service-dev.properties
```properties
# 数据库驱动：
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 数据库连接地址
spring.datasource.url=jdbc:mysql://localhost:3306/cloud-order?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
# 数据库用户名&密码：
spring.datasource.username=root
spring.datasource.password=123456
# 数据源名称
spring.datasource.name=defaultDataSource
# 配置mybatis-plus 打印sql日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
# xml文件路径
mybatis-plus.mapper-locations=classpath:/mapper/**/*.xml
# 配置mybatis-plus 包路径
mybatis-plus.type-aliases-package=com.example.order.domain
```
> `nacos`/order-service.properties
```properties
author.name=不是菜狗爱编程
author.date=2024/04/05/9:13
author.description=第一次发布
```
> 配置如下
```properties
# 应用服务 WEB 访问端口
server.port=8090
# 服务名
spring.application.name=order-service
# nacos服务发现
spring.cloud.nacos.discovery.server-addr=localhost:8848
# 配置集群名称，也就是机房位置，例如JS(江苏)
spring.cloud.nacos.discovery.cluster-name=JS
# 命名空间id
spring.cloud.nacos.discovery.namespace=d2455f6d-ed00-41ba-9915-09021bc0df6c
# 是否是临时实例
spring.cloud.nacos.discovery.ephemeral=false
# nacos服务配置
spring.cloud.nacos.config.server-addr=localhost:8848
spring.cloud.nacos.config.namespace=d2455f6d-ed00-41ba-9915-09021bc0df6c
# dataId前缀
spring.cloud.nacos.config.prefix=${spring.application.name}
# 环境 (前缀-环境.后缀 如user-dev.properties)
spring.profiles.active=dev
# dataId后缀
spring.cloud.nacos.config.file-extension=properties
spring.config.import[0]=optional:nacos:${spring.cloud.nacos.config.prefix}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}?refresh=true
spring.config.import[1]=optional:nacos:${spring.cloud.nacos.config.prefix}.${spring.cloud.nacos.config.file-extension}?refresh=true
```
> 如果使用yml的方式
```yaml
spring:
  config:
    import:
      - optional:nacos:order-service.${spring.cloud.nacos.config.file-extension}?refresh=true
      - optional:nacos:order-service-dev.${spring.cloud.nacos.config.file-extension}?refresh=true
```
#### 持久化
> 将nacos持久化到数据库
>
> **`注意`**先导出一份配置，待会持久化到mysql之后导入之前的配置
![image-20240406182620580](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406182620580.png)
> 打开`SQL`执行界面
![image-20240406182656584](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406182656584.png)
> 注意不要直接执行，需要`新建数据库`，这里命令为`nacos_config`
```sql
use nacos_config
```
![image-20240406183031988](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406183031988.png)
![image-20240406184136353](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406184136353.png)
> 注意`账号`、`密码`不要写错
![image-20240406184229297](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406184229297.png)
> 否则会报错如下
![image-20240406184247861](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406184247861.png)
> 导入之前的配置（`order-service-dev.properties`、`order-service.properties`、`user-service-dev.properties`）
![image-20240406184827019](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406184827019.png)
> 可以看到数据库里已经保存了该配置的数据
![image-20240406185007386](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406185007386.png)
## Feign
### 入门
> feign是一个`http`客户端
>
> 先来看我们以前利用RestTemplate发起远程调用的代码:
```java
User user = restTemplate.getForObject("http://user-service/user/" + orders.getUserId(), User.class);
```
存在下面的问题:
1. 代码可读性差，编程体验不统一
2. 参数复杂URL难以维护
`Feign`是一个`声明式`的`http客户端`，官方地址: https://github.com/OpenFeign/feign
其作用就是帮助我们优雅的实现http请求的发送,解决上面提到的问题。
1. 启动类上添加`@EnableFeignClients`注解
2. `@FeignClient("provider-service")`注解指定服务名，并复制`服务发布者`的controller方法到`interface`里，如
```java
@FeignClient("provider-service")
public interface ProviderClient {
    @GetMapping("/provider/{string}")
    public String echo(@PathVariable(value = "string") String string);
}
```
3. 然后注入`ProviderClient`即可使用
```java
@RestController
@RequestMapping("consumer")
public class ConsumerController {
    @Autowired
    private ProviderClient providerClient;
    @GetMapping("/{str}")
    public Map<String, Object> echo(@PathVariable String str) {
        Map<String, Object> map = new HashMap<>();
        String provider = providerClient.echo(str);
        map.put("provider",provider);
        map.put("consumer","this is consumer,"+str);
        return map;
    }
}
```
> 现在尝试一下，
>
> 原本的`RestTemplate`代码可以删掉了
```java
    //@Bean
    //@LoadBalanced
    //public RestTemplate restTemplate(){
    //    return new RestTemplate();
    //}
```
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```
```java
@EnableFeignClients
@EnableDiscoveryClient
@MapperScan("com.example.order.mapper")
@SpringBootApplication
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```
> 这里的配置需要注意一下，容易出问题
>
> `@FeignClient`指定了服务名
>
> `@GetMapping("/user/{id}")`这里一定要和`被调用`服务controller里写的一样才行，那边`User服务`类上有`@RequestMapping("user")`注解
> 所以这里`@GetMapping("/{id}")`前面一定要加`/user`
>
> 方法名，返回值一定都得一样，建议直接去复制`UserController`即可
```java
@FeignClient("user-service")
public interface UserFeignClient {
    @GetMapping("/user/{id}")
    public User findOne(@PathVariable(value = "id") Integer id);
}
```
```java
@RefreshScope
@RestController
@RequestMapping("order")
public class OrdersController {
    @Autowired
    private OrdersService ordersService;
    @Autowired
    private UserFeignClient userFeignClient;
 
    @GetMapping("/{id}")
    public Orders findOne(@PathVariable Integer id) {
        Orders orders = ordersService.getById(id);
        User user = userFeignClient.findOne(orders.getUserId());
        orders.setUser(user);
        return orders;
    }
}
```
> 这样就很优雅了
### SpringBoot3新特性
> SpringBoot3新特性内置声明式HTTP客户端，这里浅试一下
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
```
```java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
    @Bean
    public ProviderFluxClient userRestClient() {
        WebClient client = WebClient.builder()
                .baseUrl("http://localhost:8080")
                .build();
        HttpServiceProxyFactory factory = HttpServiceProxyFactory.builder(WebClientAdapter.forClient(client))
                .build();
        return factory.createClient(ProviderFluxClient.class);
    }
}
```
```java
@HttpExchange("/provider")
public interface ProviderFluxClient {
    @GetExchange("/{string}")
    Flux<String> echo(@PathVariable(value = "string") String string);
}
```
```java
@RestController
@RequestMapping("consumer")
public class ConsumerController {
    @Autowired
    private ProviderFluxClient providerFluxClient;
    @GetMapping("/{str}")
    public String echo(@PathVariable String str) {
        providerFluxClient.echo(str).subscribe(
                data -> System.out.println("data:" + data)
        );
        return "this is consumer," + str;
    }
}
```
> 还是回到主题`feign`来
### Feign自定义配置
![无标题](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/%E6%97%A0%E6%A0%87%E9%A2%98.png)
```properties
# springboot默认日志日志级别
logging.level.com.example=debug
# default是全局配置，如果写服务名称，则是针对某个微服务的配置
# 日志级别是 full
#spring.cloud.openfeign.client.config.default.logger-level=full
spring.cloud.openfeign.client.config.user-service.logger-level=full
```
> 也可以使用java代码配置
```java
import feign.Logger;
import org.springframework.context.annotation.Bean;
public class FeignClientConfiguration {
    @Bean
    public Logger.Level feignLogLevel(){
        return Logger.Level.BASIC;
    }
}
```
> 如果希望这个配置在全局生效，在启动类上配置如下
```java
@EnableFeignClients(defaultConfiguration = FeignClientConfiguration.class)
```
> 如果希望针对某个服务生效
```java
@FeignClient(value = "user-service",configuration = FeignClientConfiguration.class)
```
> 正常使用`Basic`即可，调试错误用`Full`，日志会对性能有影响
### 性能优化
Feign底层的客户端实现:
1. URLConnection:默认实现，不支持连接池
2. Apache HttpClient:支持连接池
3. OKHttp:支持连接池
添加如下依赖
```xml
        <dependency>
            <groupId>io.github.openfeign</groupId>
            <artifactId>feign-httpclient</artifactId>
            <version>9.4.0</version>
        </dependency>
```
```properties
# springboot默认日志日志级别
logging.level.com.example=debug
spring.cloud.openfeign.client.config.user-service.logger-level=basic
# 开启对httpclient的支持
spring.cloud.openfeign.httpclient.enabled=true
# 最大连接数200
spring.cloud.openfeign.httpclient.max-connections=200
# 每个路径的最大连接数
spring.cloud.openfeign.httpclient.max-connections-per-route=50
```
> 这两个值`max-connections`和`max-connections-per-route`的最佳选择还是需要使用压测工具来针对自己项目测试得出
### 最佳实践
> Feign的最佳实践
1. 方式一(继承):给消费者的FeignClient和提供者的controller定义统-的父接口作为标准。
   ![image-20240406215310739](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406215310739.png)
**缺点**
- 服务紧耦合
- 父接口参数列表中的映射不会被继承
2. 方式二( 抽取) :将FeignClient抽取为独立模块,并且把接口有关的POJO、默认的Feign配置都放到这个模块中，提供给所有消费者使用
![image-20240406215531738](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406215531738.png)
**缺点**
引用了过多的api，或许`order`只需要引入`user`的一两个`api`即可，但是现在却全部引入了
> 尝试一下方式二，现在再新建两个项目`common`和`feign-api`，将一些公共类抽取到`common`中，如实体类`User`或者`Order`
![image-20240406222235617](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406222235617.png)
> common/pom.xml
```xml
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
```
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Order {
    private Integer id;
    private String name;
    private Integer price;
    private Integer num;
    private Integer userId;
    private User user;
}
```
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer id;
    private String username;
    private String address;
}
```
> feign-api/pom.xml
>
> 导入`common`依赖
```xml
    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
    </dependencies>
```
```java
@FeignClient(value = "user-service")
public interface UserFeignClient {
    @GetMapping("/user/{id}")
    public User findOne(@PathVariable(value = "id") Integer id);
}
```
```java
import feign.Logger;
import org.springframework.context.annotation.Bean;
public class FeignClientConfiguration {
    @Bean
    public Logger.Level feignLogLevel(){
        return Logger.Level.BASIC;
    }
}
```
> order/pom.xml
```xml
    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>feign-api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>io.github.openfeign</groupId>
            <artifactId>feign-httpclient</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-loadbalancer</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>com.alibaba.nacos</groupId>
                    <artifactId>nacos-client</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.alibaba.nacos</groupId>
            <artifactId>nacos-client</artifactId>
        </dependency>
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```
```java
@EnableFeignClients(defaultConfiguration = FeignClientConfiguration.class)
@EnableDiscoveryClient
@MapperScan("com.example.order.mapper")
@SpringBootApplication
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```
```java
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.example.common.entity.User;
import com.example.feign.api.UserFeignClient;
import com.example.order.domain.Orders;
import com.example.order.service.OrdersService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.*;
import java.util.List;
@RefreshScope
@RestController
@RequestMapping("order")
public class OrdersController {
    @Autowired
    private OrdersService ordersService;
    @Autowired
    private UserFeignClient userFeignClient;
    @GetMapping("/{id}")
    public Orders findOne(@PathVariable Integer id) {
        Orders orders = ordersService.getById(id);
        User user = userFeignClient.findOne(orders.getUserId());
        orders.setUser(user);
        return orders;
    }
}
```
> 启动确报错
```
Consider defining a bean of type 'com.example.feign.api.UserFeignClient' in your configuration.
```
> 因为包扫描的问题，`order`服务的默认包扫描在`com.example.order`下，但是`feign-api`的包在`com.example.feign.api`下
![image-20240406222933521](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406222933521.png)
> 建议选第二种
```java
@EnableFeignClients(defaultConfiguration = FeignClientConfiguration.class,
        basePackageClasses = {
                UserFeignClient.class
        })
@EnableDiscoveryClient
@MapperScan("com.example.order.mapper")
@SpringBootApplication
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```
> 再次启动即可
![image-20240406223012310](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406223012310.png)
## Gateway
### 入门
为什么需要网关?
**网关功能:**
1. 身份认证和权限校验
2. 服务路由、 负载均衡
3. 请求限流
![image-20240406223620424](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240406223620424.png)
在SpringCloud中网关的实现包括两种:
1. gateway
2. zuul
Zuul是基于Servlet的实现，属于阻塞式编程。而SpringCloudGateway则是 基于Spring5中提供的WebFlux,属于响应式编程的实现，具备更好的性能。
### 项目创建
**gateway/pom.xml**
```xml
		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-loadbalancer</artifactId>
        </dependency>
		<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
```
**gateway/application.yml**
> 注意这里网关一定要和`order`、`user`服务加入到同一个`nacos命名空间`，否则会报503
>
> 而且要添加`spring-cloud-loadbalancer`的依赖，否则也会报503
```yaml
spring:
  application:
    name: gateway-service
  cloud:
    nacos:
      config:
        server-addr: localhost:8848
        namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
        import-check:
          enabled: false
      discovery:
        server-addr: localhost:8848
        namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
    gateway:
      # 网关路由配置
      routes:
        # 路由id，自定义唯一即可
        - id: user-service
          # 目标路由地址，这种方式用的较少
          # uri: http://localhost:8080
          # lb就是LoadBalancer，负载均衡的意思，后面是服务名称
          uri: lb://user-service
          # 路由断言，判断请求是否符合路由规则条件
          predicates:
            # 路径匹配。匹配以/user/开头的路由
            - Path=/user/**
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/order/**
server:
  port: 8888
```
![image-20240407071648831](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407071648831.png)
![image-20240407071705475](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407071705475.png)
> 可以看到已经成功路由到`order`服务了
>
> 整个流程如下图所示
![image-20240407071801233](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407071801233.png)
> 路由配置包括
1. 路由id:路由的唯一标示.
2. 路由目标(uri):路由的目标地址，http代表固定地址，lb代表根据服务名负载均衡
3. 路由断言(predicates):判断路由的规则,
4. 路由过滤器(filters):对请求或响应做处理
### 路由断言工厂
我们在配置文件中写的断言规则只是字符串,这些字符串会被Predicate Factory读取并处理,转变为路由判断的条件
例如`Path=/user/**`是按照路径匹配，这个规则是由`org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory`类来处理的
![image-20240407072100082](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407072100082.png)
> 详细查看[spring官网](https://spring.io/projects/spring-cloud-gateway)
>
> 这里试一下`After`，如果不知道时间格式怎么写，可以使用如下方法打印
```java
    @Test
    void contextLoads() {
        ZonedDateTime now = ZonedDateTime.now();
        System.out.println("now = " + now);
    }
```
```yaml
spring:
  application:
    name: gateway-service
  cloud:
    nacos:
      config:
        server-addr: localhost:8848
        namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
        import-check:
          enabled: false
      discovery:
        server-addr: localhost:8848
        namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
    gateway:
      # 网关路由配置
      routes:
        # 路由id，自定义唯一即可
        - id: user-service
          # 目标路由地址，这种方式用的较少
          # uri: http://localhost:8080
          # lb就是LoadBalancer，负载均衡的意思，后面是服务名称
          uri: lb://user-service
          # 路由断言，判断请求是否符合路由规则条件
          predicates:
            # 路径匹配。匹配以/user/开头的路由
            - Path=/user/**
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/order/**
            - After=2024-04-07T07:43:30.566635800+08:00[Asia/Shanghai]
server:
  port: 8888
```
> 当路由中没有匹配的路由，就会报404无法路由
### 路由过滤器
GatewayFilter是网关中提供的一种过滤器，可以对进入网关的请求和微服务返回的响应做处理
![image-20240407073828621](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407073828621.png)
![image-20240407073840487](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407073840487.png)
```yaml
spring:
  application:
    name: gateway-service
  cloud:
    nacos:
      config:
        server-addr: localhost:8848
        namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
        import-check:
          enabled: false
      discovery:
        server-addr: localhost:8848
        namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
    gateway:
      # 网关路由配置
      routes:
        # 路由id，自定义唯一即可
        - id: user-service
          # 目标路由地址，这种方式用的较少
          # uri: http://localhost:8080
          # lb就是LoadBalancer，负载均衡的意思，后面是服务名称
          uri: lb://user-service
          # 路由断言，判断请求是否符合路由规则条件
          predicates:
            # 路径匹配。匹配以/user/开头的路由
            - Path=/user/**
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/order/**
          filters:
          # 添加请求头
            - AddRequestHeader=color,this is red
server:
  port: 8888
```
```java
@RefreshScope
@RestController
@RequestMapping("order")
public class OrdersController {
    @Autowired
    private OrdersService ordersService;
    @Autowired
    private UserFeignClient userFeignClient;
    @GetMapping("/{id}")
    public Orders findOne(@PathVariable Integer id,@RequestHeader("color") String color) {
        System.out.println("color = " + color);
        Orders orders = ordersService.getById(id);
        User user = userFeignClient.findOne(orders.getUserId());
        orders.setUser(user);
        return orders;
    }
}
```
> 通过网关路由到`order`服务，`order`服务控制台打印`color = this is red`
>
> 如果希望配置所有网关都配置该过滤器，可以使用`默认过滤器`
>
> `默认过滤器`对所有路由都生效
```yaml
spring:
  application:
    name: gateway-service
  cloud:
    nacos:
      config:
        server-addr: localhost:8848
        namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
        import-check:
          enabled: false
      discovery:
        server-addr: localhost:8848
        namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
    gateway:
      # 网关路由配置
      routes:
        # 路由id，自定义唯一即可
        - id: user-service
          # 目标路由地址，这种方式用的较少
          # uri: http://localhost:8080
          # lb就是LoadBalancer，负载均衡的意思，后面是服务名称
          uri: lb://user-service
          # 路由断言，判断请求是否符合路由规则条件
          predicates:
            # 路径匹配。匹配以/user/开头的路由
            - Path=/user/**
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/order/**
      # 默认过滤器
      default-filters:
        - AddRequestHeader=color,this is red
server:
  port: 8888
```
### 全局过滤器
> 全局过滤器的作用也是处理一-切进 入网关的请求和微服务响应，与`GatewayFilter`的作用一样。
>
> 区别在于`GatewayFilter`通过配置定义，处理逻辑是固定的。而`GlobalFilter`的逻辑需要自己写代码实现。
>
> 感觉和`default-filters`很像，但是如果需要自定义一些复杂的逻辑，就得使用`GatewayFilter`了
>
> 定义方式是实现`GatewayFilter`接口
![image-20240407075426584](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407075426584.png)
```java
public interface GlobalFilter {
	/**
	 * Process the Web request and (optionally) delegate to the next {@code WebFilter}
	 * through the given {@link GatewayFilterChain}.
	 * @param exchange the current server exchange
	 * @param chain provides a way to delegate to the next filter
	 * @return {@code Mono<Void>} to indicate when request processing is complete
	 */
	Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain);
}
```
**需求**
定义全局过滤器，拦截请求，判断请求的参数是否满足下面条件:
1. 参数中是否有authorization
2. authorization参数值是否为admin
> 在`gateway`服务下新建`AuthorizationFilter`并实现`GlobalFilter`接口
>
> 整体步骤如下
```java
public class AuthorizationFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 获取请求参数
        // 获取参数中的authorization参数
        // 判断参数值是否等于admin
        // 是，则放行。否，则拦截
        return null;
    }
}
```
> 具体实现如下
```java
// 过滤器优先级，越小越高
@Order(1)
@Component
public class AuthorizationFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 获取请求参数
        ServerHttpRequest request = exchange.getRequest();
        MultiValueMap<String, String> queryParams = request.getQueryParams();
        // 获取参数中的 authorization 参数
        String authorization = queryParams.getFirst("authorization");
        // 判断参数值是否等于admin
        if("admin".equals(authorization)){
           // 是，则放行
           return chain.filter(exchange);
        }
        // 设置状态码
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        // 否，则拦截
        return exchange.getResponse().setComplete();
    }
}
```
> 除了通过`@Order`注解来定义过滤器优先级，还可以实现`Ordered`接口来重写`getOrder`方法定义过滤器优先级
```java
@Component
public class AuthorizationFilter implements GlobalFilter , Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 获取请求参数
        ServerHttpRequest request = exchange.getRequest();
        MultiValueMap<String, String> queryParams = request.getQueryParams();
        // 获取参数中的 authorization 参数
        String authorization = queryParams.getFirst("authorization");
        // 判断参数值是否等于admin
        if("admin".equals(authorization)){
           // 是，则放行
           return chain.filter(exchange);
        }
        // 设置状态码
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        // 否，则拦截
        return exchange.getResponse().setComplete();
    }
    @Override
    public int getOrder() {
        return 1;
    }
}
```
> 访问http://localhost:8888/order/1
![image-20240407080630998](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407080630998.png)
> 访问http://localhost:8888/order/1?authorization=admin
![image-20240407080653461](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407080653461.png)
### 过滤器执行顺序
> 请求进入网关会碰到三类过滤器:当前路由的过滤器、DefaultFilter. GlobalFilter
> 请求路由后，会将当前路由过滤器和DefaultFjlter、GlobalFilter, 合并到一个过滤器链(集合)中，排序后依次执行每个过滤器
>
> 在`GatewayFilterAdapter`中`GatewayFilter`被适配成了`GatewayFilter`
>
> 网关中的所有过滤器最终都是`GatewayFilter`
![image-20240407081153557](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407081153557.png)
![image-20240407081341725](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407081341725.png)
**过滤器执行顺序**
1. 每一个过滤器都必须指定一个int类型的order值， order值越小， 优先级越高，执行顺序越靠前。
2. GlobalFilter通过实现Ordered接口，或者添加@Order注解来指定order值，由我们自己指定
3. 路由过滤器和defaultFilter的order由Spring指定,默认是按照声明顺序从1递增。
4. 当过滤器的order值一样时， 会按照`defaultFilter` >`路由过滤器`> `GlobalFilter`的顺序执行。
![image-20240407081533812](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407081533812.png)
#### 跨域问题处理
> 网关是基于`WebFlux`来实现的，之前的`Servlet Api`就无法使用了
跨域:域名不一致就是跨域，主要包括:
1. 域名不同: www.taobao.com 和www.taobao.org和www.jd.com和miaosha.jd.com
2. 域名相同，端口不同: localhost:8080和localhost8081
跨域问题:浏览器禁止请求的发起者与服务端发生跨域ajax请求,请求被浏览器拦截的问题
网关处理跨域采用的同样是CORS方案,并且只需要简单配置即可实现
![image-20240407082008295](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407082008295.png)
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
<pre>
spring:
  cloud:
    gateway:
      globalcors: # 全局的跨域处理
        add-to-simple-url-handler-mapping: true # 解决options请求被拦截问题
        corsConfigurations:
          '[/**]':
            allowedOrigins: # 允许哪些网站的跨域请求
              - "http://localhost:8090"
              - "http://www.leyou.com"
            allowedMethods: # 允许的跨域ajax的请求方式
              - "GET"
              - "POST"
              - "DELETE"
              - "PUT"
              - "OPTIONS"
            allowedHeaders: "*" # 允许在请求中携带的头信息
            allowCredentials: true # 是否允许携带cookie
            maxAge: 360000 # 这次跨域检测的有效期
</pre>
</body>
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
<script>
  axios.get("http://localhost:10010/user/1?authorization=admin")
  .then(resp => console.log(resp.data))
  .catch(err => console.log(err))
</script>
</html>
```
> 使用`VS Code`启动
![image-20240407082338156](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407082338156.png)
> 配置
```yaml
spring:
  cloud:
    gateway:
      globalcors: # 全局的跨域处理
        add-to-simple-url-handler-mapping: true # 解决options请求被拦截问题
        corsConfigurations:
          '[/**]':
            allowedOrigins: # 允许哪些网站的跨域请求
              - "http://localhost:8090"
              - "http://www.leyou.com"
            allowedMethods: # 允许的跨域ajax的请求方式
              - "GET"
              - "POST"
              - "DELETE"
              - "PUT"
              - "OPTIONS"
            allowedHeaders: "*" # 允许在请求中携带的头信息
            allowCredentials: true # 是否允许携带cookie
            maxAge: 360000 # 这次跨域检测的有效期
```
> 没用，依然报错跨域问题
>
> 由于spring-framework从5.3.0版本开始，关于CORS跨域配置类 **CorsConfiguration**中将 `allowedOrigins` 变量名修改为 `allowedOriginPatterns`
>
> 所以，如果项目中 spring-framework 版本高于5.3.0，请使用如下配置代码
```yaml
spring:
  application:
    name: gateway-service
  cloud:
    nacos:
      config:
        server-addr: localhost:8848
        namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
        import-check:
          enabled: false
      discovery:
        server-addr: localhost:8848
        namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
    gateway:
      # 网关路由配置
      routes:
        # 路由id，自定义唯一即可
        - id: user-service
          # 目标路由地址，这种方式用的较少
          # uri: http://localhost:8080
          # lb就是LoadBalancer，负载均衡的意思，后面是服务名称
          uri: lb://user-service
          # 路由断言，判断请求是否符合路由规则条件
          predicates:
            # 路径匹配。匹配以/user/开头的路由
            - Path=/user/**
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/order/**
      default-filters:
        - AddRequestHeader=color,this is red
      globalcors: # 全局的跨域处理
        add-to-simple-url-handler-mapping: true # 解决options请求被拦截问题
        corsConfigurations:
          '[/**]':
            allowedOriginPatterns: # 允许哪些网站的跨域请求
              - "http://localhost:8090"
              - "http://localhost:5500"
              - "http://www.leyou.com"
            allowedMethods: # 允许的跨域ajax的请求方式
              - "GET"
              - "POST"
              - "DELETE"
              - "PUT"
              - "OPTIONS"
            allowedHeaders: "*" # 允许在请求中携带的头信息
            allowCredentials: true # 是否允许携带cookie
            maxAge: 360000 # 这次跨域检测的有效期,一定时间内不需要在验证跨域，直接访问
server:
  port: 8888
```
