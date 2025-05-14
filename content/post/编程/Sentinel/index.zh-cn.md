---
title: Sentinel
description: Sentinel
date: 2024-04-10
slug: Sentinel
image: 202412212132620.png
categories:
    - Sentinel
---
# Sentinel
## 初识Sentinel
### 雪崩问题
> 故障的服务D，会导致服务A的tomcat资源耗尽
>
> 微服务调用链路中的某个服务故障，引起整个链路中的所有微服务都不可用，这就是`雪崩`。
![image-20240403074558194](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240403074558194.png)
**解决雪崩问题的常见方式有四种:**
1. 超时处理:设定超时时间，请求超过-定时间没有响应就返回错误信息，不会无休止等待
2. 舱壁模式:限定每个业务能使用的线程数,避免耗尽整个tomcat的资源，因此也叫线程隔离。
3. 熔断降级:由断路器统计业务执行的异常比例，如果超出阈值则会熔断该业务，拦截访问该业务的一切请求。
4. 流量控制:限制业务访问的QPS，避免服务因流量的突增而故障。
### 技术选型
![image-20240403075332866](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240403075332866.png)
> 同类型的框架还有`Resilience4j`、`Polly`、`Envoy`等
### 安装Sentinel
[Sentinel下载地址](https://github.com/alibaba/Sentinel/releases)，选择`sentinel-dashboard-xxx.jar`，自己新建一个`startup.txt`
```sh
java -jar sentinel-dashboard-1.8.5.jar
```
![image-20240403080118647](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240403080118647.png)
> 然后将`startup.txt`后缀改为`bat`，双击运行
>
> 或者自定义启动端口号
```sh
java -Dserver.port=8718 -jar sentinel-dashboard-1.8.5.jar
```
![image-20240403080917682](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240403080917682.png)
> 账号密码默认是`sentinel`
![image-20240403081023548](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240403081023548.png)
**配置开机自启动**
[博客链接](https://blog.csdn.net/qq_52288743/article/details/131084881#:~:text=Win%20%2B%20R%20%E8%BE%93%E5%85%A5%E4%BB%A5%E4%B8%8B%E5%91%BD%E4%BB%A4%20%E5%9B%9E%E8%BD%A6%20shell%3ACommon%20Startup%20%E9%99%84%E5%8A%A0%EF%BC%9A%E5%A6%82%E9%9C%80%E6%9A%82%E5%81%9C%E8%87%AA%E5%90%AF%E5%8A%A8%E5%8F%AF%E4%BB%A5%E5%9C%A8%E4%BB%BB%E5%8A%A1%E7%AE%A1%E7%90%86%E5%99%A8%E8%BF%9B%E8%A1%8C%E5%85%B3%E9%97%AD,%E5%9C%A8%20%E5%90%AF%E5%8A%A8%20%E5%90%8E%EF%BC%8C%E6%82%A8%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87%E8%AE%BF%E9%97%AE%20Sentinel%20%E7%9A%84%E6%8E%A7%E5%88%B6%E5%8F%B0%E6%9D%A5%E6%9F%A5%E7%9C%8B%E7%9B%91%E6%8E%A7%E6%95%B0%E6%8D%AE%E5%92%8C%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%E3%80%82%20%E9%BB%98%E8%AE%A4%E6%83%85%E5%86%B5%E4%B8%8B%EF%BC%8C%20Sentinel%20%E7%9A%84%E6%8E%A7%E5%88%B6%E5%8F%B0%E5%9C%B0%E5%9D%80%E4%B8%BA%EF%BC%9Ahttp%3A%2F%2Flocalhost%3A8080%E3%80%82)
## 创建项目
> 在`order`服务导入`sentinel`依赖，并配置`application.properties`
> 然后访问一下`order`服务的`controller` http://localhost:8090/order/1
```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
```
```properties
# 配置控制台地址
spring.cloud.sentinel.transport.dashboard=localhost:8089
```
![image-20240407204039706](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407204039706.png)
> sentinel主要采用`线程池隔离`、`熔断降级`、`流量控制`的方式来解决雪崩问题
## 簇点链路
簇点链路:就是项目内的调用链路,链路中被监控的每个接口就是一个资源。 默认情况下sentinel会监控SpringMVC的每一个端点(Endpoint) ，因此SpringMVC的每一个端点(Endpoint)就是调用链路中的一-个资源。
流控、熔断等都是针对簇点链路中的资源来设置的，因此我们可以点击对应资源后面的按钮来设置规则:
**需求**:给/order/{orderld}这个资源设置流控规则，QPS不能超过5。然后进行压力测试。
![image-20240407205129957](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407205129957.png)
![image-20240407210436655](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407210436655.png)
![image-20240407210503016](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407210503016.png)
## 流控模式
> 在添加限流规则时，点击高级选项，可以选择三种流控模式:
1. `直接`:统计当前资源的请求，触发阈值时对当前资源直接限流，也是默认的模式
2. `关联`:统计与当前资源相关的另一一个资源，触发阈值时，对当前资源限流
3. `链路`:统计从指定链路访问到本资源的请求，触发阈值时,对指定链路限流
### 关联模式
统计与当前资源相关的另一个资源，触发阈值时，对当前资源限流
使用场景:比如用户支付时需要修改订单状态，同时用户要查询订单。查询和修改操作会争抢数据库锁,产生竞争。业务需求是有限支付和更新订单的业务,因此当修改订单业务触发阈值时，需要对查询订单业务限流，防止影响到更新订单的业务。
![image-20240407210759266](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407210759266.png)
**需求**
1. 在OrderController新建两个端点: `/order/query`和`/order/update`, 无需实现业务
2. 配置流控规则，当`/order/update`资源被访问的QPS超过5时，对`/order/query`请求限流
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
    @GetMapping("/query")
    public String query() {
        return "查询订单成功";
    }
    @GetMapping("/update")
    public String update() {
        return "更新订单成功";
    }
}
```
> 然后访问一下 http://localhost:8090/order/query 和 http://localhost:8090/order/update、
>
> 对`/order/query`添加流控
![image-20240407211433491](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407211433491.png)
![image-20240407211555637](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407211555637.png)
![image-20240407211548557](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407211548557.png)
> 对`update`压力测试，限流`query`
>
> 满足下面条件可以使用关联模式:
>
> 1. 两个有竞争关系的资源
> 2. 一个`优先级较高`，一个`优先级较低`，我们希望当`优先级高`的`触发阈值`时，对`优先级低`的进行`限流`
### 链路模式
链路模式:只针对从指定链路访问到本资源的请求做统计，判断是否超过阈值。
例如有两条请求链路:
/test1 -> /common
/test2 -> /common
如果只希望统计从/test2进入到/common的请求，则可以这样配置:
![image-20240407211931646](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407211931646.png)
**需求**:有查询订单和创建订单业务,两者都需要查询商品。针对从查询订单进入到查询商品的请求统计并设置限流。
步骤:
1. 在OrderService 中添加一个queryGoods方法，不用实现业务
2. 在OrderController中 ，改造/order/query端点，调用OrderService中的queryGoods方法
3. 在OrderController中 添加一个/order/save的端 点,调用OrderService的queryGoods方法
4. 给queryGoods设 置限流规则,从/order/query进 入queryGoods的方法限制QPS必须小于2
```java
@Service
public class OrdersServiceImpl extends ServiceImpl<OrdersMapper, Orders>
    implements OrdersService{
    @Override
    public void queryGoods() {
        System.out.println("查询商品成功");
    }
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
    @GetMapping("/query")
    public String query() {
        ordersService.queryGoods();
        return "查询订单成功";
    }
    @GetMapping("/update")
    public String update() {
        return "更新订单成功";
    }
    @GetMapping("/save")
    public String save() {
        ordersService.queryGoods();
        return "创建订单成功";
    }
}
```
> 而`service`中的`queryGoods()`并没有被`sentinel`监控，无法配置限流规则，所以应该对其进行监控
Sentinel默认只标记Controller中的方法为资源，如果要标记其它方法，需要利用`@SentinelResource`注解
```java
@Service
public class OrdersServiceImpl extends ServiceImpl<OrdersMapper, Orders>
    implements OrdersService{
    @SentinelResource("goods")
    @Override
    public void queryGoods() {
        System.out.println("查询商品成功");
    }
}
```
Sentinel默认会将Controller方法做context整合,导致链路模式的流控失效
当 Sentinel 默认开启 `web-context-unify` 时，会将所有 Controller 方法的上下文路径进行整合，这意味着所有通过 Controller 的调用都会被视作一个统一的资源路径，这可能会导致链路模式下的流控规则失效。
具体来说，假设你有一个服务内部有多个接口相互调用形成了一条调用链路，如 `/save -> /goods` 和 `/query -> /goods`，如果你想要针对 `/query ` 调用到 `/goods` 的这条特定调用链路设置流量控制，那么在 context 整合的情况下，Sentinel 无法区分这两个不同的调用链路，因为它们在上下文路径上都被视为同一个资源。
需要修改`application.properties`,添加配置
```properties
# 配置控制台地址
spring.cloud.sentinel.transport.dashboard=localhost:8089
# 关闭 context 整合
spring.cloud.sentinel.web-context-unify=false
```
> 如果不关闭`context整合`，所有的`controller`会被认为是同一个根链路发展而来的子链路
>
> 访问一下 http://localhost:8090/order/query 和 http://localhost:8090/order/save
![image-20240407212903207](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407212903207.png)
![image-20240407213029296](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407213029296.png)
> 删除其他的流控规则
![image-20240407213116501](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407213116501.png)
> 使用`Jmeter`进行压力测试
**/order/save**
![image-20240407213258792](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407213258792.png)
**/order/query**
![image-20240407213305172](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240407213305172.png)
**总结**
直接:对当前资源限流
关联:高优先级资源触发阈值，对低优先级资源限流。
链路:阈值统计时，只统计从指定资源进入当前资源的请求是对请求来源的限流
## 流控效果
### 概念
流控效果是指请求达到流控阈值时应该采取的措施，包括三种:
1. `快速失败`:达到阈值后，新的请求会被立即拒绝并拋出FlowException异常。是默认的处理方式。
2. `warm up`:预热模式，对超出阈值的请求同样是拒绝并抛出异常。但这种模式阈值会动态变化，从一-个较小值逐渐增加到最大阈值。
3. `排队等待`:让所有的请求按照先后次序排队执行，两个请求的间隔不能小于指定时长
### 预热模式warm up
warm up也叫预热模式，是应对服务冷启动的一种方案。`threshold`为最大阈值，`coldFactor`为冷启动因子，请求阈值初始值是`threshold/coldFactor`,持续指定时长后，逐渐提高到`threshold`值。而`coldFactor`的默认值是3.
例如，我设置QPS的`threshold`为10，预热时间为5秒,那么`初始阈值`就是`10/3`，也就是3,然后在5秒后逐渐增长到10
![image-20240408064325063](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408064325063.png)
**需求**
给`/order/{id}`这个资源设置限流,最大QPS为10,利用warm up效果,预热时长为5秒
![image-20240408065105430](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408065105430.png)
![image-20240408065329162](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408065329162.png)
![image-20240408065457513](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408065457513.png)
![image-20240408065517390](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408065517390.png)
### 排队等待
当请求超过QPS阈值时，快速失败和warm up会拒绝新的请求并抛出异常。而排队等待则是让所有请求进入一个队列中,然后按照阈值允许的时间间隔依次执行。后来的请求必须等待前面执行完成，如果请求预期的等待时间超出最大时长,则会被拒绝。
例如: QPS=5,意味着每`200ms`处理一个队列中的请求; timeout = 2000， 意味着预期等待超过2000ms的请求会被拒绝并抛出异常
![image-20240408065759385](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408065759385.png)
![image-20240408065827751](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408065827751.png)
**需求**
给`/order/{id}`这个资源设置限流，最大QPS为10， 利用排队的流控效果，超时时长设置为5s
![image-20240408070509839](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408070509839.png)
> 而`Jmeter`配置的`QPS`是15，如果按照之前的情况（直接失败），则会有`5`个`QPS`无法处理
![image-20240408070703747](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408070703747.png)
![image-20240408070729937](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408070729937.png)
> 到后面请求超时则会直接失败
![image-20240408070746902](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408070746902.png)
![image-20240408070809963](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408070809963.png)	
### 总结
流控效果有哪些？
1. 快速失败: QPS超过阈值时，拒绝新的请求
2. warm up: QPS超过阈值时，拒绝新的请求; QPS阈值是逐渐提升的，可以避免冷启动时高并发导致服务宕机。
3. 排队等待:请求会进入队列，按照阈值允许的时间间隔依次执行请求;如果请求预期等待时长大于超时时间，直接拒绝
## 热点参数限流
> 之前的限流是统计访问某个资源的所有请求，判断是否超过QPS阈值。而热点参数限流是分别统计`参数值相同`的请求,判断是否超过QPS阈值。
![image-20240408071109096](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408071109096.png)
> **案例**
>
> 给`/order/{id}`这个资源添加热点参数限流，规则如下:
>
> 1. 默认的热点参数规则是每1秒请求量不超过`2`
> 2. 给1这个参数设置例外:每1秒请求量不超过`4`
> 3. 给2这个参数设置例外:每1秒请求量不超过`10`
>
> 需要`注意`的是`热点参数限流`对默认的SpringMVC资源无效，只有通过`@SentinelResource`注解声明的资源才能生效
```java
@RefreshScope
@RestController
@RequestMapping("order")
public class OrdersController {
    @Autowired
    private OrdersService ordersService;
    @Autowired
    private UserFeignClient userFeignClient;
    @SentinelResource("hot")
    @GetMapping("/{id}")
    public Orders findOne(@PathVariable Integer id) {
        Orders orders = ordersService.getById(id);
        User user = userFeignClient.findOne(orders.getUserId());
        orders.setUser(user);
        return orders;
    }
}
```
![image-20240408072009011](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408072009011.png)
> 可以看到`/order/1`的`QPS`是4
![image-20240408072731930](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408072731930.png)
>可以看到`/order/2`的`QPS`是10，全部通过
![image-20240408072739726](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408072739726.png)
>可以看到`/order/3`的`QPS`是默认阈值，即`2`
![image-20240408072747154](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408072747154.png)
## 隔离和降级
> **虽然限流可以尽量避免因高并发而引起的服务故障，但服务还会因为其它原因而故障。而要将这些故障控制在一定范围，避免雪崩，就要靠线程隔离(舱壁模式)和熔断降级手段了。**
>
> 不管是线程隔离还是熔断降级,都是对`客户端(调用方，示例中的服务A)`的保护。
![image-20240408073136305](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240408073136305.png)
SpringCloud中，微服务调用都是通过Feign来实现的，因此做客户端保护必须整合Feign和Sentinel。
1. 修改OrderService的`application.properties`文件，开启Feign的Sentinel功能
```properties
feign.sentinel.enabled=true
```
   
2. 给FeignClient编写失败后的降级逻辑，但调用的服务异常时，返回`备用`方案
   - 方式一: FallbackClass,无法对远程调用的异常做处理
   - 方式二: FallbackFactory,可以对远程调用的异常做处理，我们选择这种
3. 在`feign-api`项目中定义类，实现`FallbackFactory`
> 不少教程导入的是`hystrix`包下的`FallbackFactory`，但是`sentinel`已经不再使用`hystrix`依赖了，而`openfeign`包下也有一个`FallbackFactory`
>
> 不确定是不是这个原因导致后文报错
```java
@Slf4j
public class UserClientFallbackFactory implements FallbackFactory<UserFeignClient> {
    @Override
    public UserFeignClient create(Throwable cause) {
        return new UserFeignClient() {
            @Override
            public User findOne(Integer id) {
                log.info("findOne方法异常，执行降级逻辑");
                return new User();
            }
        };
    }
}
```
2. 在`feign-api`项目中将`UserClientFallbackFactory`注册为一个Bean:
这里没有加`@Configuration`注解是因为在`OrderApplication`配置了`@EnableFeignClients`
```java
public class FeignClientConfiguration {
    @Bean
    public Logger.Level feignLogLevel() {
        return Logger.Level.BASIC;
    }
    @Bean
    public UserClientFallbackFactory userClientFallbackFactory() {
        return new UserClientFallbackFactory();
    }
}
```
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
3. 在`feign-api`项目中的`UserClient`接口中使用`UserClientFallbackFactory`:
```java
@FeignClient(value = "user-service",fallbackFactory = UserClientFallbackFactory.class)
public interface UserFeignClient {
    @GetMapping("/user/{id}")
    public User findOne(@PathVariable(value = "id") Integer id);
}
```
```java
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'consumerController': Unsatisfied dependency expressed through field 'providerClient': Error creating bean with name 'feignSentinelBuilder' defined in class path resource [com/alibaba/cloud/sentinel/feign/SentinelFeignAutoConfiguration.class]: Post-processing of merged bean definition failed
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'feignSentinelBuilder' defined in class path resource [com/alibaba/cloud/sentinel/feign/SentinelFeignAutoConfiguration.class]: Post-processing of merged bean definition failed
Caused by: java.lang.IllegalStateException: Failed to introspect Class [com.alibaba.cloud.sentinel.feign.SentinelFeign$Builder] from ClassLoader [jdk.internal.loader.ClassLoaders$AppClassLoader@63947c6b]
Caused by: java.lang.NoClassDefFoundError: org/springframework/cloud/openfeign/FeignClientFactory
Caused by: java.lang.ClassNotFoundException: org.springframework.cloud.openfeign.FeignClientFactory
	at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:641) ~[na:na]
	at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:188) ~[na:na]    
```
> 这里整合feign各种报错，估计是版本原因
>
> 我尝试使用`2.3.12.RELEASE`的springboot是可以实现`feign + nacos + sentinel`的(还是jdk17)
>
> demo链接放到github上了，[链接](https://github.com/IsUnderAchiever/sentinel-demo)
>
> 做了个`springboot3、nacos、openfeign、sentinel`的demo，还是上面的这个链接，请切换`boot3`版本
>
> springboot3报错解决，不使用`UserClientFallbackFactory`，删除该文件，`feign.sentinel.enabled`配置也删除
>
> 使用`@SentinelResource`配合`blockHandler`属性
**在服务被调用方新增**
```java
@RestController
@RequestMapping("provider")
public class ProviderController {
    @SentinelResource(value = "providerQuery",blockHandler = "blockHandler")
    @GetMapping("/query")
    public String query(){
        return "query...";
    }
    public static String blockHandler(BlockException e) {
        return "请稍后再试！";
    }
}
```
**添加流控规则**
![image-20240410075355925](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410075355925.png)
**访问provider服务**
![image-20240410075433734](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410075433734.png)
> 访问`provider`自然是没有问题的，关键是`consumer`那边怎么样
![image-20240410075510088](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410075510088.png)
> 可以，也被限流了
>
> 需要注意的是`blockHandler`指定的方法返回值要和`被标记@SentinelResource`的方法`(这里是query)`保持一致，得是`String`
>
> 否则会报错如下
```
com.alibaba.csp.sentinel.slots.block.flow.FlowException: null
```
> **我们希望能同意来配置**，而不是每个类都写一个单独的限流方法
```java
/**
 * 全局限流处理
 * @author: 不是菜狗爱编程
 * @date: 2024/04/10/7:27
 * @description:
 */
public class GlobeBlockException {
    public static String blockHandler(BlockException e) {
        return "您访问太频繁了，请稍后再试！";
    }
}
```
```java
@RestController
@RequestMapping("provider")
public class ProviderController {
    @SentinelResource(value = "providerQuery",blockHandlerClass = GlobeBlockException.class, blockHandler = "blockHandler")
    @GetMapping("/query")
    public String query(){
        return "query...";
    }
}
```
> 以上代码在github中，[链接](https://github.com/IsUnderAchiever/sentinel-demo)
>
> **注意切换boot3分支，main分支使用的是2.3.12.RELEASE版本springboot，`2.3.12.RELEASE版本springboot`可以正常使用自定义`FallbackFactory`类**
>
> 以上是关于大概的解法，下面展示`order`和`user`模块的写法
**user模块**
> 注意这里设置的`blockHandler`方法参数要比`@SentinelResource`标记的方法参数多一个`BlockException`，其它参数保持一致
```java
public class GlobeBlockException {
    public static User blockHandler(@PathVariable Integer id,BlockException e) {
        return new User();
    }
}
```
```java
@RestController
@RequestMapping("user")
public class UserController {
    @Autowired
    private UserService userService;
    @SentinelResource(value = "userQueryById",blockHandlerClass = GlobeBlockException.class,blockHandler = "blockHandler")
    @GetMapping("/{id}")
    public User findOne(@PathVariable Integer id) {
        return userService.getById(id);
    }
}
```
![image-20240410081054018](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410081054018.png)
> 又出现一个问题，springboot启动多个user实例，但是sentinel设置限流规则后只对其中一个实例有效
![image-20240410081722803](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410081722803.png)
![image-20240410081732670](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410081732670.png)
> 看`sentinel`界面
>
> 在这里设置多个即可
![image-20240410081809138](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410081809138.png)
![image-20240410081841313](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410081841313.png)
> order也有效
![image-20240410081915014](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410081915014.png)
## 线程隔离
> 左侧是`线程隔离`，右侧是`信号量隔离`
![image-20240409073319242](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240409073319242.png)
> 对比
![image-20240409073605422](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240409073605422.png)
> 将原本的`QPS`换成`线程数`即可
**需求**
给UserClient的查询用户接口设置流控规则，线程数不能超过2。然后利用jemeter测试。
![image-20240410082119553](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410082119553.png)
![image-20240410082637264](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410082637264.png)
> 这里的请求都是正常，因为之前配置了降级的逻辑，所以会直接返回一个`空的user对象`，而不是报错
## 熔断降级
### 概念
熔断降级是解决雪崩问题的重要手段。其思路是由断路器统计服务调用的异常比例、慢请求比例，如果超出阈值则会熔断该服务。即拦截访问该服务的一切请求;而当服务恢复时,断路器会放行访问该服务的请求。
![image-20240409074149240](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240409074149240.png)
### 熔断策略--慢调用
断路器熔断策略有三种:`慢调用`、`异常比例`、`异常数`
慢调用:业务的响应时长(RT)大于指定时长的请求认定为慢调用请求。在指定时间内，如果请求数量超过设定的最小数量，慢调用比例大于设定的阈值，则触发熔断。例如:
![image-20240409074310444](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240409074310444.png)
**解读**: RT超过500ms的调用是慢调用，统计最近10000ms内的请求，如果请求量超过10次,并且慢调用比例不低于0.5则触发熔断，熔断时长为5秒。然后进入half-open状态,放行一次请求做测试。
**需求**
> 给UserClient的查询用户接口设置降级规则，慢调用的RT阈值为50ms,统计时间为1秒，最小请求数量为5，失败阈值比例为0.4,熔断时长为5
>
> 为了增加业务耗时，可以设置`Thread.sleep(60)`
![image-20240410083103666](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410083103666.png)
```java
@RestController
@RequestMapping("user")
public class UserController {
    @Autowired
    private UserService userService;
    @SentinelResource(value = "userQueryById",blockHandlerClass = GlobeBlockException.class,blockHandler = "blockHandler")
    @GetMapping("/{id}")
    public User findOne(@PathVariable Integer id) {
        try {
            Thread.sleep(60);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return userService.getById(id);
    }
}
```
> 此时我们正常的响应时间是`71ms`
![image-20240410083240454](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410083240454.png)
> 快速访问多次后熔断，熔断之后已经不再去尝试了，而是直接返回失败结果，这里的时间为`4ms`
![image-20240410083454593](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410083454593.png)
> 服务的调用方响应如下
![image-20240410083623355](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410083623355.png)
> 针对多实例也有效
![image-20240410084150507](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410084150507.png)
### 熔断策略--异常比例、异常数
异常比例或异常数:统计指定时间内的调用，如果调用次数超过指定请求数，并且出现异常的比例达到设定的比例阈值(或超过指定异常数) ,则触发熔断。例如:
![image-20240409074723270](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240409074723270.png)
![image-20240409074746502](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240409074746502.png)
**解读**:统计最近1000ms内的请求，如果请求量超过10次，并且异常比例不低于0.5,则触发熔断，熔断时长为5秒。然后进入half-open状态，放行一次 请求做测试。
> 故意抛出异常即可
![image-20240410210541628](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410210541628.png)
```java
@RestController
@RequestMapping("user")
public class UserController {
    @Autowired
    private UserService userService;
    @SentinelResource(value = "userQueryById",blockHandlerClass = GlobeBlockException.class,blockHandler = "blockHandler")
    @GetMapping("/{id}")
    public User findOne(@PathVariable Integer id) {
        if(id==2){
            throw new RuntimeException("抛出异常了");
        }
        return userService.getById(id);
    }
}
```
![image-20240410210729986](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410210729986.png)
> 如果觉得直接返回报错不太友好，可以使用`@RestControllerAdvice`来进行统一的异常处理，这里不过多介绍这个注解，先看看熔断是否生效
![](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410210916311.png)
> 再看看`order`服务调用是否正常
![image-20240410211017304](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410211017304.png)
### 总结
Sentinel熔断降级的策略有哪些?
慢调用比例:超过指定时长的调用为慢调用，统计单位时长内慢调用的比例，超过阈值则熔断
异常比例:统计单位时长内异常调用的比例，超过阈值则熔断
异常数:统计单位时长内异常调用的次数,超过阈值则熔断
## 授权规则
> 当时在`gateway`网关的时候有做过`过滤`，可以阻止某些请求通过`网关`路由到`微服务`
>
> 但如果有一天`微服务`的地址及端口被泄露出去了，请求就可能不通过网关，直接访问服务的的地址，这个时候`微服务`就很危险了
授权规则可以对调用方的来源做控制，有白名单和黑名单两种方式。
白名单:来源(origin) 在白名单内的调用者允许访问
黑名单:来源(origin)在黑名单内的调用者不允许访问
![image-20240409075531220](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240409075531220.png)
例如，我们限定只允许从网关来的请求访问`order-service`，不允许浏览器来直接访问,那么流控应用中就填写网关的名称.
这里的`流控应用`其实不是指的`网关服务`的名称，而是指的`orign`请求来源的名称
Sentinel是通过`RequestOriginParser`这个接口的parseOrigin来获取请求的来源的。
而`sentinel`默认情况下，从`gateway`和`浏览器`到达的`orign`名称都是`default`，无法对`网关`和`浏览器进行区分`
所以需要实现`RequestOriginParser`该接口来自定义实现，比如
```java
@Component
public class HeaderOriginParser implements RequestOriginParser {
    @Override
    public String parse0rigin(HttpServletRequest request) {
        String origin = request. getHeader("origin") ;
        if(Str ingUtils. isEmpty(origin)){
            return "blank";
        }
        return origin;
	}
}
```
> 但现实是`网关`和`浏览器`默认都没有`origin`请求头，而我们给`网关`来的请求添加上`origin`即可
>
> 网关过滤器`AddRequestHeader`可以实现，这里直接写成`默认过滤器`，所有经过`网关`的请求都添加上`origin`头
```yaml
spring:
    cloud:
        gateway:
            default-filters:
                #添加名为origin的请求头，值为gateway
                -AddRequestHeader=origin,gateway 
```
> 所以`流控应用`就填`gateway`，注意这里的值，即`gateway`不要暴露了
**理论结束，实践开始**
> order服务
```java
@Slf4j
@Component
public class HeadOriginParser implements RequestOriginParser {
    @Override
    public String parseOrigin(HttpServletRequest httpServletRequest) {
        // 获取请求头
        String origin = httpServletRequest.getHeader("origin");
        // 判空
        if(!StringUtils.hasLength(origin)){
            // origin为空
            origin="blank";
        }
        log.info("当前请求:{},origin请求头:{}",httpServletRequest.getRequestURI(),origin);
        return origin;
    }
}
```
> 网关服务配置，主要是`- AddRequestHeader=origin,gateway`
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
#        - AddRequestHeader=color,this is red
        - AddRequestHeader=origin,gateway
```
![image-20240410220549522](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410220549522.png)
> 通过网关服务访问
![image-20240410220636398](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410220636398.png)
> 直接访问
![image-20240410220652041](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410220652041.png)
## 自定义异常结果
默认情况下，发生限流、降级、授权拦截时，都会抛出异常到调用方。如果要自定义异常时的返回结果，需要实现`BlockExceptionHandler`接口
![image-20240409081053921](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240409081053921.png)
> 判断这里异常的类型，然后返回自定义结果
**而`BlockException`包含很多个子类，分别对应不同的场景:**
![image-20240409081138373](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240409081138373.png)
> 具体写法可参考
![image-20240409081246546](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240409081246546.png)
> order服务
```java
@Component
public class SentinelBlockHandler implements BlockExceptionHandler {
    @Override
    public void handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, BlockException e) throws Exception {
        String msg = "未知异常";
        int status = 429;
        if (e instanceof FlowException) {
            msg = "请求被限流了! ";
        } else if (e instanceof DegradeException) {
            msg = "请求被降级了! ";
        } else if (e instanceof ParamFlowException) {
            msg = "热点参数限流! ";
        } else if (e instanceof AuthorityException) {
            msg = "请求没有权限! ";
            status = 401;
        }
        httpServletResponse.setContentType("application/json;charset=utf-8");
        httpServletResponse.setStatus(status);
        httpServletResponse.getWriter().println("{\"message\": \"" + msg + "\"， \"status\": " + status + "}");
    }
}
```
> 重启服务后新建授权规则
![image-20240410221758019](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410221758019.png)
> 也可以使用`@SentinelResource`注解方式
```java
@RestController
@RequestMapping("order")
public class OrdersController {
    @Autowired
    private OrdersService ordersService;
    @SentinelResource(value = "orderFindOne",blockHandlerClass = GlobeBlockException.class,blockHandler = "blockHandler")
    @GetMapping("/{id}")
    public Orders findOne(@PathVariable Integer id) {
        return ordersService.queryByOrderId(id);
    }
}
```
```java
@Slf4j
public class GlobeBlockException {
    public static Orders blockHandler(@PathVariable Integer id, BlockException e) {
        log.error("当前服务繁忙，请稍后再试");
        return new Orders();
    }
}
```
> 配置授权规则
![image-20240410222328041](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410222328041.png)
> 虽然只能返回与使用`@SentinelResource`注解标注的方法`相同`的`返回值类型`，但是在`springboot`的`restful`风格的api中，我们通常会统一返回结果
> 如`JsonData`或者`R`等统一返回值类型
## 规则持久化
### 概念
Sentinel的控制台规则管理有三种模式:
1. 原始模式: Sentinel的默认模式，将规则保存在内存,重启服务会丢失。
2. pull模式
3. push模式
#### pull模式
pull模式:控制台将配置的规则推送到Sentinel客户端,而客户端会将配置规则保存在本地文件或数据库中。以后会定时去本地文件或数据库中查询，更新本地规则。
![image-20240409081715561](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240409081715561.png)
> 由于是定时轮询读取，所以存在`时效性`问题，导致了服务中`规则`不一致的问题
#### push模式
push模式:控制台将配置规则推送到远程配置中心，例如Nacos。Sentinel客户 端监听Nacos,获取配置变更的推送消息，完成本地配置更新。
![image-20240409081844429](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240409081844429.png)
#### 总结
Sentinel的三种配置管理模式是什么?
1. 原始模式:保存在内存
2. pull模式:保存在本地文件或数据库，定时去读取
3. push模式:保存在nacos，监听变更实时更新
### 实现push模式
> 参考[博客](https://blog.csdn.net/qq_36763419/article/details/121560105)
push模式实现最为复杂，依赖于nacos,并且需要修改Sentinel控制台源码。
**引入依赖**
```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-sentinel-datasource</artifactId>
</dependency>
```
**配置nacos地址**
> 示例如下
```yaml
spring:
    cloud:
        sentinel:
            datasource:
                flow:
                    nacos:
                    	# nacos地址
                        server-addr: localhost:8848 
                        dataId: orderservice-flow-rules
                        groupId: SENTINEL_GROUP
                        #还可以是: degrade(降级)、authority(授权)、 param-flow(参数限流)
                        rule-type: flow 
				# 如果需要多个配置，继续添加即可，如下所示
                degrade:
                    nacos:
                        server-addr: localhost:8848 
                        dataId: orderservice-degrade-rules
                        groupId: SENTINEL_GROUP
                        rule-type: degrade 
```
>以上是配置格式，接下来正式配置`user`
```yaml
server:
  port: 8080
spring:
  application:
    name: user-service
  cloud:
    nacos:
      config:
        file-extension: properties
        namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
        prefix: ${spring.application.name}
        server-addr: localhost:8848
      discovery:
        cluster-name: JS
        ephemeral: false
        namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
        server-addr: localhost:8848
    sentinel:
      transport:
        dashboard: localhost:8089
      web-context-unify: false
      datasource:
        flow:
          nacos:
            server-addr: localhost:8848
            namespace: d2455f6d-ed00-41ba-9915-09021bc0df6c
            data-id: user-service-sentinel
            group-id: DEFAULT_GROUP
            data-type: json
            #还可以是: degrade(降级)、authority(授权)、 param-flow(参数限流)
            rule-type: flow
  config:
    import: nacos:${spring.cloud.nacos.config.prefix}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}?refresh=true
  profiles:
    active: dev
```
> 上面配置中，sentinel规则配置到nacos dataId为`user-service-sentinel`的配置中了
```json
[
  {
    "resource": "userQueryById",
    "controlBehavior": 0,
    "count": 5,
    "grade": 1,
    "limitApp": "default",
    "strategy": 0
  }
]
```
![image-20240410214414971](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410214414971.png)
> 以上配置是指
>
> resource：资源名。
> limitApp：来源应用。
> grade：阈值类型。0 表示线程数，1 表示是QPS。
> count：单机阈值。
> strategy：流控模式。0 表示直接，1 表示关联，2 表示链路。
> controlBehavior：流控效果。0 表示快速失败，1 表示Warm up，2 表示排队等待。
> clusterMode：是否集群。false 表示否，true 表示是。
**这里选择`resource`为`userQueryById`**，因为`@SentinelResource`标记资源为`userQueryById`还记得吗？
```java
@RestController
@RequestMapping("user")
public class UserController {
    @Autowired
    private UserService userService;
    @SentinelResource(value = "userQueryById",blockHandlerClass = GlobeBlockException.class,blockHandler = "blockHandler")
    @GetMapping("/{id}")
    public User findOne(@PathVariable Integer id) {
        if(id==2){
            throw new RuntimeException("抛出异常了");
        }
        return userService.getById(id);
    }
}
```
> 测试一下是否可行
![image-20240410214733973](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410214733973.png)
> order也没问题
![image-20240410214813076](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410214813076.png)
> 重启服务，查看`sentinel`控制台，规则是否还存在
![image-20240410214914582](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240410214914582.png)
> 但是有个需要注意的点，这是`sentinel`读`nacos`配置的规则，而在`sentinel`中配置的规则，不能同步到`nacos`
>
> 如果需要让`sentinel`的配置同步到`nacos`，`nacos`修改的配置`sentinel`也能获取到，则需要修改`sentinel-dashboard.jar`的源码了
>
> 网上也有教程
