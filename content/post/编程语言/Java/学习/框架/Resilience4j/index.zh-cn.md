---
title: Resilience4j
description: Resilience4j
date: 2024-04-15
slug: Resilience4j
image: 202412212128271.png
categories:
    - Resilience4j
---
# Resilience4j
## 项目搭建
> 熔断限流框架
```xml
    <properties>
        <spring-boot.version>3.0.2</spring-boot.version>
        <spring-cloud.version>2022.0.0-RC2</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.30</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
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
```
> 可直接导入`spring-cloud-starter-circuitbreaker-resilience4j`依赖或者导入`resilience4j-spring-boot3`
>
> `aop`的依赖一定要导入
```xml
        <dependency>
            <groupId>io.github.resilience4j</groupId>
            <artifactId>resilience4j-spring-boot3</artifactId>
            <version>2.2.0</version>
        </dependency>
```
```yaml
server:
  port: 8080
spring:
  application:
    name: resilience4j-demo
# 配置Resilience4j
resilience4j:
  # 重试机制
  retry:
    # 定义多个重试策略实例
    instances:
      # 充实策略实例名称
      retryApi:
        # 最大重试次数
        max-attempts: 3
        # 每次重试等待1s
        wait-duration: 1s
  # 断路器
  circuitbreaker:
    # 定义多个断路器实例
    instances:
      # 第一个断路器实例名称
      circuitBreakerApi:
        # 健康监测
        register-health-indicator: true
        # 滑动窗口大小
        sliding-window-size: 10
        # 但断路器处于半开状态时，允许的最大调用次数为3
        permitted-number-of-calls-in-half-open-state: 3
        # 滑动窗口类型
        sliding-window-type: time_based
        # 断路器打开的最小请求数
        minimum-number-of-calls: 5
        # 断路器打开的时间
        wait-duration-in-open-state: 5s
        # 失败率达到20%时，断路器打开
        failure-rate-threshold: 20
        # 事件缓冲区大小
        event-consumer-buffer-size: 10
  # 配置限流
  ratelimiter:
    # 定义多个限流策略实例
    instances:
      # 第一个限流策略实例名称
      flowLimitApi:
        # 在一个特定的时间周期内，允许的最大请求数量为（QPS为1）
        limit-for-period: 1
        # 这个时间周期的长度是1秒，即每1秒会重置请求计数
        limit-refresh-period: 1s
        # 当请求超过限制时，客户端应立即收到超时的响应，而不等待处理
        timeout-duration: 100ms
```
> 配置`RestTemplate`，待会测试失败重试机制，我这里就直接放在启动类下了
```java
@SpringBootApplication
public class Resilience4jDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(Resilience4jDemoApplication.class, args);
    }
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```
> 统一返回格式
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class JsonData {
    /**
     * 状态码 0 表示成功，1表示处理中，-1表示失败
     */
    private Integer code;
    /**
     * 数据
     */
    private Object data;
    /**
     * 描述
     */
    private String msg;
    // 成功，传入数据
    public static JsonData buildSuccess() {
        return new JsonData(0, null, null);
    }
    // 成功，传入数据
    public static JsonData buildSuccess(Object data) {
        return new JsonData(0, data, null);
    }
    // 失败，传入描述信息
    public static JsonData buildError(String msg) {
        return new JsonData(-1, null, msg);
    }
    // 失败，传入描述信息,状态码
    public static JsonData buildError(String msg, Integer code) {
        return new JsonData(code, null, msg);
    }
}
```
> 新建`TestController`
```java
import com.example.utils.JsonData;
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import io.github.resilience4j.ratelimiter.annotation.RateLimiter;
import io.github.resilience4j.retry.annotation.Retry;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import java.util.Random;
@Slf4j
@RestController
public class TestController {
    private RestTemplate restTemplate;
    @Autowired
    public TestController(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
    /**
     * 测试重试
     *
     * @return {@link JsonData}
     */
    @Retry(name = "retryApi",fallbackMethod = "fallback")
    @GetMapping("/test1")
    public JsonData test1() {
        log.info("test1 received");
        // 访问一个不存在的地址
        String forObject = restTemplate.getForObject("http://localhost:8085/test1", String.class);
        return JsonData.buildSuccess("test1");
    }
    /**
     * 测试熔断
     *
     * @return {@link JsonData}
     */
    @CircuitBreaker(name = "circuitBreakerApi",fallbackMethod = "fallback")
    @GetMapping("/test2")
    public JsonData test2() {
        int i = new Random().nextInt(100);
        // 40%的可能报异常，超过20%则熔断
        if(i>60){
            throw new RuntimeException("Unexpected Exception");
        }
        return JsonData.buildSuccess("test2");
    }
    /**
     * 测试限流
     *
     * @return {@link JsonData}
     */
    @RateLimiter(name = "flowLimitApi",fallbackMethod = "fallback")
    @GetMapping("/test3")
    public JsonData test3() {
        return JsonData.buildSuccess("test3");
    }
    /**
     * fallback
     *
     * @param throwable 可投掷
     * @return {@link JsonData}
     */
    private JsonData fallback(Throwable throwable){
        log.error("fallback:",throwable);
        return JsonData.buildError(throwable.getMessage());
    }
}
```
## 重试测试
![image-20240415082517797](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240415082517797.png)
> 查看日志
![image-20240415082547641](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240415082547641.png)
## 熔断测试
![image-20240415082635565](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240415082635565.png)
## 限流测试
> 快速访问
![image-20240415082659223](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240415082659223.png)
