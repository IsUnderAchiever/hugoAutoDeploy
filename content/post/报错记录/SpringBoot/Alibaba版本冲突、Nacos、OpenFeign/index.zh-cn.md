---
title: Alibaba版本冲突、Nacos、OpenFeign
description: Alibaba版本冲突、Nacos、OpenFeign
date: 2023-01-20
slug: Alibaba版本冲突、Nacos、OpenFeign
image: 202412212133331.png
categories:
    - Java
---

> 之前一直在做谷粒商城，确实学到了不少知识点，还有黑马的一个微服务全栈课程（哔哩哔哩有）
> 感觉学了不少，`eruka、nacos、feign、rabbitmq、es`
> 但是吧，一直往下学，感觉之前学过的知识点又不太清晰了
> 就准备了这个小项目试试水，结果...泪目了
## springboot、springcloud、springcloud alibaba版本不匹配
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202123336.png)
> 这个错误仅代表其中一种，意思就是版本不匹配
> 大部分的错误都是版本不匹配、冲突造成
> 如`java.lang.AbstractMethodError: null`、等
> 折磨了我半天
> 在博客上找了好多所谓的`“毕业版本依赖关系(推介使用)”`
> 然后就是不停的报`java.lang.AbstractMethodError: null`这个错
> 网上说是由于版本不匹配造成，项目根本跑不起来
github上有版本说明，[详情查看](https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E)
| Spring Cloud Alibaba Version      | Spring Cloud Version        | Spring Boot Version |
| --------------------------------- | --------------------------- | ------------------- |
| 2021.0.4.0*                  | Spring Cloud 2021.0.4 | 2.6.11              |
| 2021.0.1.0                   | Spring Cloud 2021.0.1 | 2.6.3               |
| 2021.1                       | Spring Cloud 2020.0.1 | 2.4.2           |
| 2.2.10-RC1*                       | Spring Cloud Hoxton.SR12    | 2.3.12.RELEASE      |
| `2.2.9.RELEASE`                   | `Spring Cloud Hoxton.SR12`  | `2.3.12.RELEASE`    |
| 2.2.8.RELEASE                     | Spring Cloud Hoxton.SR12    | 2.3.12.RELEASE      |
| 2.2.7.RELEASE                     | Spring Cloud Hoxton.SR12    | 2.3.12.RELEASE      |
| 2.2.6.RELEASE                     | Spring Cloud Hoxton.SR9     | 2.3.2.RELEASE       |
| 2.2.1.RELEASE                     | Spring Cloud Hoxton.SR3     | 2.2.5.RELEASE       |
| 2.2.0.RELEASE                     | Spring Cloud Hoxton.RELEASE | 2.2.X.RELEASE       |
| 2.1.4.RELEASE                     | Spring Cloud Greenwich.SR6  | 2.1.13.RELEASE      |
| 2.1.2.RELEASE                     | Spring Cloud Greenwich      | 2.1.X.RELEASE       |
| 2.0.4.RELEASE(停止维护，建议升级) | Spring Cloud Finchley       | 2.0.X.RELEASE       |
| 1.5.1.RELEASE(停止维护，建议升级) | Spring Cloud Edgware        | 1.5.X.RELEASE       |
| Spring Cloud Alibaba Version                              | Sentinel Version | Nacos Version | RocketMQ Version | Dubbo Version | Seata Version |
| --------------------------------------------------------- | ---------------- | ------------- | ---------------- | ------------- | ------------- |
| 2.2.10-RC1                                                | 1.8.6            | 2.2.0         | 4.9.4            | ~             | 1.6.1         |
| 2022.0.0.0-RC1                                            | 1.8.6            | 2.2.1-RC      | 4.9.4            | ~             | 1.6.1         |
| `2.2.9.RELEASE`                                           | 1.8.5            | `2.1.0`       | 4.9.4            | ~             | 1.5.2         |
| 2021.0.4.0                                                | 1.8.5            | 2.0.4         | 4.9.4            | ~             | 1.5.2         |
| 2.2.8.RELEASE                                             | 1.8.4            | 2.1.0         | 4.9.3            | ~             | 1.5.1         |
| 2021.0.1.0                                                | 1.8.3            | 1.4.2         | 4.9.2            | ~             | 1.4.2         |
| 2.2.7.RELEASE                                             | 1.8.1            | 2.0.3         | 4.6.1            | 2.7.13        | 1.3.0         |
| 2.2.6.RELEASE                                             | 1.8.1            | 1.4.2         | 4.4.0            | 2.7.8         | 1.3.0         |
| 2021.1 or 2.2.5.RELEASE or 2.1.4.RELEASE or 2.0.4.RELEASE | 1.8.0            | 1.4.1         | 4.4.0            | 2.7.8         | 1.3.0         |
| 2.2.3.RELEASE or 2.1.3.RELEASE or 2.0.3.RELEASE           | 1.8.0            | 1.3.3         | 4.4.0            | 2.7.8         | 1.3.0         |
| 2.2.1.RELEASE or 2.1.2.RELEASE or 2.0.2.RELEASE           | 1.7.1            | 1.2.1         | 4.4.0            | 2.7.6         | 1.2.0         |
| 2.2.0.RELEASE                                             | 1.7.1            | 1.1.4         | 4.4.0            | 2.7.4.1       | 1.0.0         |
| 2.1.1.RELEASE or 2.0.1.RELEASE or 1.5.1.RELEASE           | 1.7.0            | 1.1.4         | 4.4.0            | 2.7.3         | 0.9.0         |
| 2.1.0.RELEASE or 2.0.0.RELEASE or 1.5.0.RELEASE           | 1.6.3            | 1.1.1         | 4.4.0            | 2.7.3         | 0.7.1         |
> 这里我用的是`2.3.12.RELEASE`的springboot、`Hoxton.SR12`的springcloud、`2.2.9.RELEASE`的springcloud alibaba，但是有个需要注意的地方，nacos的版本需要在`2.1.0`，我以前用的nacos是1.4.1，报错`nacos版本不匹配`，[详情查看](https://blog.csdn.net/chenlengshao/article/details/124649870)
```error
com.alibaba.nacos.api.exception.NacosException: Request nacos server failed: 
```
这里我找了好多博文，发现了这样一篇，[原文连接](https://saint.blog.csdn.net/article/details/125039660)
```xml
<dependencyManagement>
        <dependencies>
            <!--spring boot-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud alibaba-->
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
我一直以来并不是这种写法，而是如下写法
```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.12.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```
第一次发现还有这种写法，记录一下
## nacos动态刷新--空指针
> 详情参考这篇博客[在nacos中实现自动热部署用@RefreshScope出现空指针异常。](https://blog.csdn.net/weixin_51472505/article/details/124007165)
今天使用nacos的时候发现了这个问题
只有配置中心出现了问题
```java
@RefreshScope
@RestController
@RequestMapping("/user")
public class UserController {
    @Value("${app.user.name}")
    private String name;
    @Value("${app.user.gender}")
    private String gender;
    @Value("${app.user.age}")
    private Integer age;
    @Autowired
    private UserService userService;
    @GetMapping("/list")
    public R listUser(){
        return R.ok().put("data",userService.list());
    }
    @GetMapping("/{id}")
    private R getUser(@PathVariable("id") Integer id){
        return R.ok().put("data",userService.getOne(new QueryWrapper<User>().eq("id",id)));
    }
    @GetMapping("/test1")
    private R test1(){
        return R.ok().put("name",name).put("gender",gender).put("age",age);
    }
}
```
> 就这么一个简单的controller，添加`@RefreshScope注解`之前都没问题
> 加了之后`user/{id}`就开始报空指针了
[小泽不会Java](https://blog.csdn.net/weixin_51472505)大佬是这么解释的：@RefreshScope他的默认代理方式是[CGLIB](https://so.csdn.net/so/search?q=CGLIB&spm=1001.2101.3001.7020)，但是spring中默认的代理也是CGLIB，就相当于它被代理了两次，这样可能就会导致数据消失
> 解决方法1：将`@RefreshScope`改为`@RefreshScope(proxyMode = ScopedProxyMode.DEFAULT)`
>
> 解决方法2：不使用`@RefreshScope`，使用`@ConfigurationProperties`注解
>
> `注意`：不是使用`@ConfigurationProperties`注解来替换`@RefreshScope`，详细请看下文
```properties
app.user.name=深海
app.user.age=22
app.user.gender=男
```
```java
// 配置类
@Data
@Component
@ConfigurationProperties(prefix = "app.user")
public class PatternProperties {
    private String name;
    private String gender;
    private Integer age;
}
```
```java
@PropertySource("classpath:test.properties")
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private PatternProperties properties;
    @Autowired
    private UserService userService;
    @GetMapping("/test1")
    private R test1(){
        return R.ok().put("name",properties.getName()).put("gender",properties.getGender()).put("age",properties.getAge());
    }
}
```
## 人人开源-代码生成--500
> 老实说，这些`谷粒商城`的视频里都有，时间太久，我有些记不清了
![2-1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202124459.png)
![2-2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202124956.png)
> 在url地址栏输入localhost:80，之后回车，变成了localhost，报500
> 看到了这样一篇[博客](https://www.renren.io/detail/13080)
> 和博主是一样的问题
> `解决方法`：http://localhost:/index.html
![2-3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202124874.png)
## 人人开源-代码生成--serviceimpl报错
![3-1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202124091.png)
看到了一篇博客，说是导包的问题
![3.2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202124174.png)
> 我看了又看，包没有什么问题，并不是这个原因
>
> 接下来又看到了这篇[博客](https://blog.csdn.net/m0_65832230/article/details/125896675)的p17，是一个大佬记录自己的谷粒商城踩坑记录
>
> 第17条，`serviceImpl 分页代码报错问题，那是因为你copy代码的时候去的是renrengenerator模块，应该去renrenfast 里面copy。`
>
> 恍然大悟，确实是这样，解决方法就是去renren-fast里复制
>
> 以前没注意到这个问题，没踩过这个坑，现在终于补齐了...
## open feign返回值为null
![4-1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202124059.png)
![4-2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202124374.png)
![4-3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202124910.png)
> feign的调用是成功了一半，可以看到user服务已经查询到了，但奇怪的是vacc服务没接收到
>
> 上网搜了一下为null的情况，[查看](https://blog.csdn.net/jbossjf/article/details/122275897)
>
> 对比了一下自己的代码，`@EnableFeignClients`、`@EnableDiscoveryClient`、`@FeignClient("vaccine-user")`
>
> 注解该在的都在，不是这个问题
>
> `返回数据中多对一实体中还包含了一对多的关系也会返回null`，我之前数据库里确实有写一对多的数据
> 本来是想试验一下来着，但仔细一看代码，不是这个问题
>
> `错误原因`：返回值的问题，被调用的方法的返回值是`renren-fast`的`R`，但`clients接口`的返回值是User
> 这个就是纯纯粗心了，太憨了
![4-4](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202124984.png)
## 排除mybatis-plus相关依赖
![5](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202124662.png)
> 这是由于导入了mybatis-plus相关依赖却没有配置数据库的原因
> 但是网关服务本就不需要配置数据库
> 所以解决方法就是把相关依赖排除在外
```java
@EnableDiscoveryClient
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class VaccineGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(VaccineGatewayApplication.class, args);
    }
}
```
