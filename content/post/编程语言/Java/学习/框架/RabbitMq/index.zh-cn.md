---
title: RabbitMq
description: RabbitMq
date: 2024-04-03
slug: RabbitMq
image: 202412212126390.png
categories:
    - RabbitMq
---

# RabbitMQ
> 以下具体内容参考自黑马程序员课程`微服务开发框架SpringCloud+RabbitMQ+Docker+Redis+搜索+分布式微服务全技术栈课程`
## 初识MQ
### 同步调用
微服务间基于Feign的调用就属于同步方式，存在一些问题。业务代码耦合严重，除此之外对性能影响也很严重，因为需要等待被调用的服务返回结果。
不仅如此，而且如果被调用的服务中，有某个服务挂了，则整个服务调用都会出现问题。
![image-20240402071726650](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402071726650.png)
### 异步调用
异步调用常见实现就是事件驱动模式
![image-20240402071919653](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402071919653.png)
**优势**
> 前三个解决了同步调用的问题
1. 服务解耦，现在只需要发送成功事件即可，不需要更改业务代码
2. 性能提高，吞吐量升高，
3. 服务没有强依赖，无需担心级联失败问题
4. 流量削峰，broker能起到缓冲的作用，右侧的服务则按照自身能力来获取broker的事件
   ![image-20240402072322477](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402072322477.png)
   **劣势**
> 异步通信的缺点
1. 依赖于Broker的可靠性、安全性、吞吐能力
2. 架构复杂了，业务没有明显的流程线，不好追踪管理
### MQ选型
![image-20240402073029943](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402073029943.png)
**安装步骤省略**
rabbitmq的管理界面的端口为15672
http://localhost:15672
管理员的账号和密码是`guest`
#### Overview
![image-20240402073427083](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402073427083.png)
>`Overview`是总览，可以看到MQ的节点信息，当前为单节点运行，并不存在集群
#### Connections
![image-20240402073613643](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402073613643.png)
> 无论是消息的`发布者`还是消息的`消费者`都需要与MQ建立连接
#### Channels
![image-20240402073847983](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402073847983.png)
> `Channels`是通道，在建立连接后需要创建`Channels`通道，然后消息的`发布者`和消息的`消费者`才能基于`Channels`完成消息的`发布`和`消费`
#### Exchanges
![image-20240402074101842](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402074101842.png)
> `Exchanges`交换机用于接收来自生产者的消息，将它们推入队列
#### Queues
![image-20240402074226842](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402074226842.png)
> `Queues`队列用来做消息存储
#### Admin
![image-20240402074543152](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402074543152.png)
#### MQ架构
![image-20240402074637327](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402074637327.png)
### 常见消息模型
> `基本消息队列`和`工作消息队列`没有使用到交换机，属于简单队列模型
1. 基本消息队列(BasicQueue)
2. 工作消息队列(WorkQueue)
3. 发布订阅( Publish、Subscribe )，又根据交换机类型不同分为三种
   1. Fanout Exchange:广播
   2. Direct Exchange:路由
   3. Topic Exchange:主题
## 基本消息队列
> 这里建立的项目结构如下
```
rabbitmq-demo
├─consumer
│  └─src
│      ├─main
│      │  └─resources
│      └─test
└─publisher
    └─src
        ├─main
        │  ├─java
        │  └─resources
        └─test
```
![image-20240402080719815](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402080719815.png)
> 父级pom
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>rabbitmq-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>rabbitmq-demo</name>
    <description>rabbitmq-demo</description>
    <packaging>pom</packaging>
    <modules>
        <module>consumer</module>
        <module>publisher</module>
    </modules>
    <properties>
        <java.version>17</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>3.0.2</spring-boot.version>
    </properties>
    <dependencyManagement>
        <dependencies>
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
> publisher
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>rabbitmq-demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <artifactId>publisher</artifactId>
    <name>publisher</name>
    <description>publisher</description>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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
        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
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
> consumer与publisher一样，改名字即可
>
> 发送和监听消息的示例代码如下
>
> 需要注意的是：`RabbitMQ`的端口是`5672`，`管理页面`端口才是`15672`，这里端口需要写`5672`
```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import org.junit.jupiter.api.Test;
import java.io.IOException;
import java.util.concurrent.TimeoutException;
/**
 * RabbitMQ发送消息测试
 *
 * @author: 不是菜狗爱编程
 * @date: 2024/04/02/8:11
 * @description:
 */
class SendMessageTest {
    @Test
    void sendMessageTest() throws IOException, TimeoutException {
        // 1.建立连接
        ConnectionFactory factory = new ConnectionFactory();
        // 1.1.设置连接参数，分别是：主机名、端口号、vhost、用户名、密码
        factory.setHost("localhost");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("tong");
        factory.setPassword("123456");
        // 1.2.建立连接
        Connection connection = factory.newConnection();
        // 2.创建通道Channel
        Channel channel = connection.createChannel();
        // 3.创建队列
        String queueName = "simple.queue";
        channel.queueDeclare(queueName, false, false, false, null);
        // 4.发送消息
        String message = "hello, rabbitmq!";
        channel.basicPublish("", queueName, null, message.getBytes());
        System.out.println("发送消息成功：【" + message + "】");
        // 5.关闭通道和连接
        channel.close();
        connection.close();
    }
}
```
```java
import com.rabbitmq.client.*;
import org.junit.jupiter.api.Test;
import java.io.IOException;
import java.util.concurrent.TimeoutException;
/**
 * RabbitMQ监听消息测试
 *
 * @author: 不是菜狗爱编程
 * @date: 2024/04/02/8:11
 * @description:
 */
class ListenMessageTest {
    @Test
    void listenMessageTest() throws IOException, TimeoutException {
        // 1.建立连接
        ConnectionFactory factory = new ConnectionFactory();
        // 1.1.设置连接参数，分别是：主机名、端口号、vhost、用户名、密码
        factory.setHost("localhost");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("tong");
        factory.setPassword("123456");
        // 1.2.建立连接
        Connection connection = factory.newConnection();
        // 2.创建通道Channel
        Channel channel = connection.createChannel();
        // 3.创建队列
        String queueName = "simple.queue";
        channel.queueDeclare(queueName, false, false, false, null);
        // 4.订阅消息
        channel.basicConsume(queueName, true, new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body) throws IOException {
                // 5.处理消息
                String message = new String(body);
                System.out.println("接收到消息：【" + message + "】");
            }
        });
        System.out.println("等待接收消息。。。。");
    }
}
```
**基本消息队列的消息发送流程:**
1. 建立connection
2. 创建channel
3. 利用channel声明队列
4. 利用channel向队列发送消息
**基本消息队列的消息接收流程:**
1. 建立connection
2. 创建channel
3. 利用channel声明队列
4. 定义consumer的消费行为handleDelivery()
5. 利用channel将消费者与队列绑定
## SpringAMQP
![image-20240402082842181](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402082842181.png)
特征
1. 侦听器容器，用异步处理入站消息
2. 用于发送和接收消息的RabbitTemplate
3. RabbitAdmin用于自动声明队列，交换机和绑定关系
案例流程如下
1. 在父工程中引入spring-amqp的依赖
2. 在publisher服务中利用RabbitTemplate发送消息到simple.queue这个队列
3. 在consumer服务中编写消费逻辑，绑定simple.queue这个队列
```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.IOException;
import java.util.concurrent.TimeoutException;
/**
 * RabbitMQ发送消息测试
 *
 * @author: 不是菜狗爱编程
 * @date: 2024/04/02/8:11
 * @description:
 */
@SpringBootTest
class SendMessageTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Test
    void sendMessageWithRabbitMqTemplateTest(){
        String queueName="simple.queue";
        String message="你好，这是【RabbitTemplate】发送的消息";
        rabbitTemplate.convertAndSend(queueName,message);
    }
}
```
>application.properties
```properties
# 主机名 192.168.XXX.XXX
spring.rabbitmq.host=localhost
# 端口
spring.rabbitmq.port=5672
# 虚拟主机
spring.rabbitmq.virtual-host=/
spring.rabbitmq.username=tong
spring.rabbitmq.password=123456
```

![image-20240402210813058](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402210813058.png)

> 点击`simple.queue`

![image-20240402210928426](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402210928426.png)

> consumer工程的properties也同样配置
```java
@Slf4j
@Component
public class RabbitMqListener {
    /**
     * 简单队列侦听器
     * 消息发送者发送的是字符串，所以这边也适用字符串来接收
     * @param message 消息
     */
    @RabbitListener(queues = {"simple.queue"})
    public void simpleQueueListener(String message){
        log.info("接收到的消息为:{}",message);
    }
}
```
> 启动consumer工程后接收到消息

![image-20240402211454812](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402211454812.png)

> 发现`消息消失了`，因为这是MQ的机制，`阅后即焚`，消息不可重复消费
>
## 工作消息队列
![image-20240402211730081](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402211730081.png)
> 两个消费者共同消费生产者发布的消息，由于消息`阅后即焚`的特性，所以只能消费者1消费一部分消息，消费者2消费一部分消息
> 但是当发布的消息过多，两个消费者的能力不足，则多余的消息就会堆积在`队列`里
> 当`队列`存储的`消息`达到上限时，则会发生消息丢失
>
> **`工作队列`**，可以提高消息处理速度，避免队列消息堆积
**实现思路如下**
1. 在publisher服务中定义测试方法，每秒产生50条消息，发送到simple.queue的
2. 在consumer服务中定义两个消息监听者，都监听simple.queue队列
3. 消费者1每秒处理50条消息，消费者2每秒处理10条消息
所以照理来说，消费者应该能在`1s`内`消费完`消息
```java
@SpringBootTest
class SendMessageTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    /**
     * 使用工作队列测试发送消息
     *
     * @throws InterruptedException 中断异常
     */
    @Test
    void sendMessageWithWorkQueueTest() throws InterruptedException {
        String queueName="simple.queue";
        String message="你好，这是消息";
        for (int i = 0; i < 50; i++) {
            rabbitTemplate.convertAndSend(queueName,message+i);
            Thread.sleep(20);
        }
    }
}
```
```java
@Slf4j
@Component
public class RabbitMqListener {
    /**
     * 工作队列侦听器1
     *
     * @param message 消息
     */
    @RabbitListener(queues = {"simple.queue"})
    public void workQueueListener1(String message) throws InterruptedException {
        log.info("工作队列侦听器1:接收到的消息为:{}",message);
        Thread.sleep(20);
    }
    /**
     * 工作队列侦听器2
     *
     * @param message 消息
     */
    @RabbitListener(queues = {"simple.queue"})
    public void workQueueListener2(String message) throws InterruptedException {
        log.info("工作队列侦听器2:接收到的消息为:{}",message);
        Thread.sleep(200);
    }
}
```
> 打印日志如下
```
2024-04-02T21:31:56.431+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息0
2024-04-02T21:31:56.456+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息1
2024-04-02T21:31:56.518+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息3
2024-04-02T21:31:56.580+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息5
2024-04-02T21:31:56.641+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息2
2024-04-02T21:31:56.641+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息7
2024-04-02T21:31:56.705+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息9
2024-04-02T21:31:56.767+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息11
2024-04-02T21:31:56.830+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息13
2024-04-02T21:31:56.845+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息4
2024-04-02T21:31:56.893+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息15
2024-04-02T21:31:56.956+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息17
2024-04-02T21:31:57.018+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息19
2024-04-02T21:31:57.049+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息6
2024-04-02T21:31:57.081+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息21
2024-04-02T21:31:57.145+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息23
2024-04-02T21:31:57.208+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息25
2024-04-02T21:31:57.253+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息8
2024-04-02T21:31:57.269+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息27
2024-04-02T21:31:57.333+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息29
2024-04-02T21:31:57.397+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息31
2024-04-02T21:31:57.458+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息10
2024-04-02T21:31:57.460+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息33
2024-04-02T21:31:57.522+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息35
2024-04-02T21:31:57.584+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息37
2024-04-02T21:31:57.651+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息39
2024-04-02T21:31:57.662+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息12
2024-04-02T21:31:57.710+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息41
2024-04-02T21:31:57.775+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息43
2024-04-02T21:31:57.837+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息45
2024-04-02T21:31:57.867+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息14
2024-04-02T21:31:57.900+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息47
2024-04-02T21:31:57.963+08:00  INFO 9600 --- [ntContainer#1-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器1:接收到的消息为:你好，这是消息49
2024-04-02T21:31:58.070+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息16
2024-04-02T21:31:58.276+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息18
2024-04-02T21:31:58.481+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息20
2024-04-02T21:31:58.687+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息22
2024-04-02T21:31:58.889+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息24
2024-04-02T21:31:59.096+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息26
2024-04-02T21:31:59.301+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息28
2024-04-02T21:31:59.507+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息30
2024-04-02T21:31:59.712+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息32
2024-04-02T21:31:59.917+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息34
2024-04-02T21:32:00.119+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息36
2024-04-02T21:32:00.321+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息38
2024-04-02T21:32:00.525+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息40
2024-04-02T21:32:00.730+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息42
2024-04-02T21:32:00.946+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息44
2024-04-02T21:32:01.150+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息46
2024-04-02T21:32:01.353+08:00  INFO 9600 --- [ntContainer#0-1] c.e.consumer.listener.RabbitMqListener   : 工作队列侦听器2:接收到的消息为:你好，这是消息48
```
> 根据日志打印的时间得知，消息处理的时间为`5s`，但我们预计的时间是`1s`
>
> 这是因为由于rabbitmq的`预取机制`，会在消息消费前先平分获取到消息，但消息被消费者A获取后，其他的消费者就无法再消费这些消息了
>
> 所以这里可以看到`工作队列侦听器1`和`工作队列侦听器2`都各自消费了`25`条消息，且`工作队列侦听器1`消费的是`奇数`，`工作队列侦听器2`则是`偶数`
>
> 添加`如下配置`即可解决
```properties
# 应用服务 WEB 访问端口
server.port=8090
# 主机名 192.168.XXX.XXX
spring.rabbitmq.host=localhost
# 端口
spring.rabbitmq.port=5672
# 虚拟主机
spring.rabbitmq.virtual-host=/
spring.rabbitmq.username=tong
spring.rabbitmq.password=123456
# 每次只取一条消息，处理完成再获取下一条
spring.rabbitmq.listener.simple.prefetch=1
```
## 发布、订阅
### 概念
> 发布订阅模式与之前案例的区别就是允许将同一消息发送给多个消费者。实现方式是加入了exchange (交换机)。

![image-20240402214519822](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402214519822.png)

常见exchange类型包括:
1. Fanout: 广播
2. Direct: 路由
3. Topic: 话题
**`注意`**: exchange负责消息路由，而不负责存储，路由失败则消息丢失
### Fanout 发布订阅
> Fanout Exchange会将接收到的消息路由到每一个跟其绑定的queue

![image-20240402214633520](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402214633520.png)

**实现思路如下:**
1. 在consumer服务中，利用代码声明队列、交换机，并将两者绑定
2. 在consumer服务中，编写两个消费者方法，分别监听`fanout.queue1`和`fanout.queue2`
3. 在publisher中编写测试方法，向`fanout.exchange`发送消息
```java
/**
 * FanoutExchange配置
 *
 * @author: 不是菜狗爱编程
 * @date: 2024/04/02/21:49
 * @description:
 */
@Configuration
public class FanoutExchangeConfig {
    /**
     * fanout交换机
     *
     * @return {@link FanoutExchange}
     */
    @Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanout.exchange");
    }
    /**
     * fanout队列1
     *
     * @return {@link Queue}
     */
    @Bean
    public Queue fanoutQueue1() {
        return new Queue("fanoutQueue1");
    }
    /**
     * fanout队列2
     *
     * @return {@link Queue}
     */
    @Bean
    public Queue fanoutQueue2() {
        return new Queue("fanoutQueue2");
    }
    /**
     * 绑定队列1
     *
     * @param fanoutQueue1   扇出队列1
     * @param fanoutExchange 扇出交换
     * @return {@link Binding}
     */
    @Bean
    public Binding buildingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }
    /**
     * 绑定队列2
     *
     * @param fanoutQueue2   扇出队列1
     * @param fanoutExchange 扇出交换
     * @return {@link Binding}
     */
    @Bean
    public Binding buildingQueue2(Queue fanoutQueue2, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
    }
}
```
![image-20240402215642007](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402215642007.png)
![image-20240402215706608](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402215706608.png)
> 配置`交换机`和`队列`以及它们的`绑定关系`
```java
@Configuration
public class FanoutExchangeConfig {
    /**
     * fanout交换机
     *
     * @return {@link FanoutExchange}
     */
    @Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanout.exchange");
    }
    /**
     * fanout队列1
     *
     * @return {@link Queue}
     */
    @Bean
    public Queue fanoutQueue1() {
        return new Queue("fanoutQueue1");
    }
    /**
     * fanout队列2
     *
     * @return {@link Queue}
     */
    @Bean
    public Queue fanoutQueue2() {
        return new Queue("fanoutQueue2");
    }
    /**
     * 绑定队列1
     *
     * @param fanoutQueue1   扇出队列1
     * @param fanoutExchange 扇出交换
     * @return {@link Binding}
     */
    @Bean
    public Binding buildingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }
    /**
     * 绑定队列2
     *
     * @param fanoutQueue2   扇出队列1
     * @param fanoutExchange 扇出交换
     * @return {@link Binding}
     */
    @Bean
    public Binding buildingQueue2(Queue fanoutQueue2, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
    }
}
```
```java
@Slf4j
@Component
public class RabbitMqListener {
    /**
     * fanout队列侦听器1
     *
     * @param message 消息
     */
    @RabbitListener(queues = {"fanoutQueue1"})
    public void fanoutQueueListener1(String message){
        log.info("fanout队列侦听器1:接收到的消息为:{}",message);
    }
    /**
     * fanout队列侦听器2
     *
     * @param message 消息
     */
    @RabbitListener(queues = {"fanoutQueue2"})
    public void fanoutQueueListener2(String message){
        log.info("fanout队列侦听器2:接收到的消息为:{}",message);
    }
}
```
```java
@SpringBootTest
class SendMessageTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    /**
     * 使用Fanout交换机测试发送消息
     *
     * @throws InterruptedException 中断异常
     */
    @Test
    void sendMessageWithFanoutExchangeTest() throws InterruptedException {
        String exchangeName="fanout.exchange";
        String message="你好，这是消息";
        for (int i = 0; i < 50; i++) {
            rabbitTemplate.convertAndSend(exchangeName,"",message+i);
            Thread.sleep(20);
        }
    }
}
```
### DirectExchange
> Direct Exchange会将接收到的消息根据规则路由到指定的Queue,因此称为路由模式(routes) 。
>
> 每一个Queue都与Exchange设置一个BindingKey
>
> 发布者发送消息时，指定消息的RoutingKey
>
> Exchange将消息路由到BindingKey与消息RoutingKey一致的队列

![image-20240402220448895](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402220448895.png)

**`需要注意`**：一个队列，可以绑定多个`key`
```java
@Slf4j
@Component
public class RabbitMqListener {
   
	/**
     * direct队列侦听器1
     *
     * @param message 消息
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue("directQueue1"),
            exchange = @Exchange(value = "direct.exchange",type = ExchangeTypes.DIRECT),
            key = {"red","blue"}
    ))
    public void directQueueListener1(String message){
        log.info("direct队列侦听器1:接收到的消息为:{}",message);
    }
	/**
     * direct队列侦听器2
     *
     * @param message 消息
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue("directQueue2"),
            exchange = @Exchange(value = "direct.exchange",type = ExchangeTypes.DIRECT),
            key = {"yellow","blue"}
    ))
    public void directQueueListener2(String message){
        log.info("direct队列侦听器2:接收到的消息为:{}",message);
    }
}
```
> 启动`consumer`项目，前往rabbitmq网页。发现`direct.exchange`交换机和`directQueue1`、`directQueue2`队列已经声明了
>
> 查看绑定关系
![image-20240402222040723](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240402222040723.png)

```java
@SpringBootTest
class SendMessageTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    /**
     * 使用Direct交换机测试发送消息
     *
     * @throws InterruptedException 中断异常
     */
    @Test
    void sendMessageWithDirectExchangeTest(){
        String exchangeName = "direct.exchange";
        String[] routingKeys={"yellow","red","blue"};
        for (String routingKey : routingKeys) {
            rabbitTemplate.convertAndSend(exchangeName, routingKey, "消息是:"+routingKey);
        }
    }
}
```
> 日志打印如下
```
direct队列侦听器2:接收到的消息为:消息是:yellow
direct队列侦听器1:接收到的消息为:消息是:red
direct队列侦听器1:接收到的消息为:消息是:blue
direct队列侦听器2:接收到的消息为:消息是:blue
```
### TopicExchange
> `TopicExchange`与DirectExchange类似， 区别在于routingKey必须是多个单词的列表,并且以`.`分割。
>
> Queue与Exchange指定`BindingKey`时可以使用通配符:
> `#`:代指0个或多个单词
> `*`:代指一个单词

![image-20240403065247801](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240403065247801.png)

**实现思路如下**

1. 并利用@RabbitListener声明Exchange、Queue、RoutingKey
2. 在consumer服务中，编写两个消费者方法,分别监听`topic.queue1`和`topic.queue2`
3. 在publisher中编写测试方法，向`topic.exchange`发送消息
```java
/**
 * RabbitMQ 侦听器
 *
 * @author: 不是菜狗爱编程
 * @date: 2024/04/02/21:11
 * @description:
 */
@Slf4j
@Component
public class RabbitMqListener {
    /**
     * 主题队列侦听器1
     * 接收 china 的一切信息
     *
     * @param message 消息
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue("topicQueue1"),
            exchange = @Exchange(value = "topic.exchange",type = ExchangeTypes.TOPIC),
            key = "china.#"
    ))
    public void topicQueueListener1(String message){
        log.info("topic队列侦听器1:接收到的消息为:{}",message);
    }
    /**
     * 主题队列侦听器2
     * 接收一切 news 信息
     *
     * @param message 消息
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue("topicQueue2"),
            exchange = @Exchange(value = "topic.exchange",type = ExchangeTypes.TOPIC),
            key = "#.news"
    ))
    public void topicQueueListener2(String message){
        log.info("topic队列侦听器2:接收到的消息为:{}",message);
    }
}
```
启动项目后，确认绑定关系
![image-20240403065809837](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240403065809837.png)
```java
/**
 * RabbitMQ发送消息测试
 *
 * @author: 不是菜狗爱编程
 * @date: 2024/04/02/8:11
 * @description:
 */
@SpringBootTest
class SendMessageTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    /**
     * 使用Topic交换机测试发送消息
     */
    @Test
    void sendMessageWithTopicExchangeTest(){
        String exchangeName = "topic.exchange";
        String[] routingKeys={"china.weather","china.news","Japan.weather","Japan.news"};
        for (String routingKey : routingKeys) {
            rabbitTemplate.convertAndSend(exchangeName, routingKey, "消息是:"+routingKey);
        }
    }
}
```
## 消息转换器
> 说明:在SpringAMQP的发送方法中，接收消息的类型是Object,也就是说我们可以发送任意对象类型的消息，SpringAMQP会帮我们序列化为字节后发送。
```java
@Configuration
public class FanoutExchangeConfig {
    
    /**
     * 对象队列
     *
     * @return {@link Queue}
     */
    @Bean
    public Queue objectQueue(){
        return new Queue("objectQueue");
    }
}
```
```java
@Slf4j
@Component
public class RabbitMqListener {
    @RabbitListener(queues = {"objectQueue"})
    public void objectQueueListener(Object object){
        log.info("objectQueue队列侦听器:接收到的消息为:{}",object);
    }
}
```
> 这里将consumer启动之后，就会创建`objectQueue`队列
```java
@SpringBootTest
class SendMessageTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    /**
     * 发送对象消息测试
     */
    @Test
    void sendObjectMessageTest() {
        String queueName="objectQueue";
        Map<String, Object> map = new HashMap<>();
        map.put("name","马小跳");
        map.put("gender","男");
        map.put("age","18");
        rabbitTemplate.convertAndSend(queueName,map);
    }
}
```
> 消费消息的日志打印如下
```
objectQueue队列侦听器:接收到的消息为:(Body:'[serialized object]' MessageProperties [headers={}, contentType=application/x-java-serialized-object, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=, receivedRoutingKey=objectQueue, deliveryTag=1, consumerTag=amq.ctag-6jbSNzh0393W9MP8GZ1z-w, consumerQueue=objectQueue])
```
> 这次将`consumer`项目关闭，我们重新发送信息，去`rabbitmq`管理页面看一看消息

![image-20240403071928081](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240403071928081.png)

> rabbitmq原生只支持字节，但是spring允许我们发送对象，这是因为使用了jdk序列化，但是这种序列化有缺点
>
> 1. 性能较差
> 2. 安全性问题，容易出现注入问题
> 3. 数据长度太长了（消息体越大，传输消息的速度越慢，而且占用额外的内存空间）
>
> `如何重新设定序列化的方式？`
>
> Spring的对消息对象的处理是由`org.springframework.amqp.support.converter.MessageConverter`来处理的。
>
> 而默认实现是`SimpleMessageConverter`,基于JDK的ObjectOutputStream完成序列化。
>
> 如果要修改只需要定义一个MessageConverter类型的Bean即可。推荐用JSON方式序列化
```xml
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
```
```java
@Configuration
public class RabbitMqConfig {
    /**
     * 采用json的方式将消息序列化
     *
     * @return {@link MessageConverter}
     */
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```
> 重新发送消息，前往rabbitmq管理页面查看

![image-20240403072815201](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240403072815201.png)

> 消息消费也需要配置`jackson`依赖、`RabbitMqConfig`，上面代码复制粘贴即可，这里不再重复
```java
/**
 * RabbitMQ 侦听器
 *
 * @author: 不是菜狗爱编程
 * @date: 2024/04/02/21:11
 * @description:
 */
@Slf4j
@Component
public class RabbitMqListener {
    
    /**
     * 对象队列侦听器
     *
     * @param map 对象
     */
    @RabbitListener(queues = {"objectQueue"})
    public void objectQueueListener(Map<String,Object> map){
        log.info("objectQueue队列侦听器:接收到的消息为:{}",map);
        for (Map.Entry<String, Object> entry : map.entrySet()) {
            System.out.println("entry = " + entry);
        }
    }
}
```
**总结**
SpringAMQP中消息的序列化和反序列化是怎么实现的?
1. 利用MessageConverter实现的，默认是JDK的序列化
2. 注意发送方与接收方必须使用相同的MessageConverter
