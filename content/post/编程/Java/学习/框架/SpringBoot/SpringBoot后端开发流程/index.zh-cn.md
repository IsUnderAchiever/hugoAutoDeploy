---
title: SpringBoot后端开发流程
description: SpringBoot后端开发流程
date: 2023-03-13
slug: SpringBoot后端开发流程
image: 202412212133331.png
categories:
    - Spring
---

SpringBoot后端开发流程
===============
> jwt生成以及解析token、拦截器、全局异常处理、自定义参数解析、跨域、参数校验
依赖
-----------------------------------------
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.12.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>springbootTest</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootTest</name>
    <description>springbootTest</description>
    <properties>
        <java.version>1.8</java.version>
        <jjwt.version>0.9.1</jjwt.version>
        <fastjson.version>1.2.72</fastjson.version>
        <mybatis-plus.version>3.5.1</mybatis-plus.version>
    </properties>
    <dependencies>
        <!--jsr303-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/cn.easyproject/orai18n -->
        <!-- 如果数据库选用mysql则不需要此依赖，oracle则需要此依赖，不然后面会报错 -->
        <dependency>
            <groupId>cn.easyproject</groupId>
            <artifactId>orai18n</artifactId>
            <version>12.1.0.2.0</version>
        </dependency>
        <!--mybatis plus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus.version}</version>
        </dependency>
        <!--jjwt-->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>${jjwt.version}</version>
        </dependency>
        <!--web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--json-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${fastjson.version}</version>
        </dependency>
        <!--oracle-->
        <dependency>
            <groupId>com.oracle.database.jdbc</groupId>
            <artifactId>ojdbc8</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
```sql
create table tb_user
(
    id        number(10) primary key,
    nickname  varchar2(20),
    username  varchar2(20),
    password varchar2(20)
);
insert into tb_user (id, nickname, username, password) values (1,'哈哈','admin','123456');
insert into tb_user (id, nickname, username, password) values (2,'呵呵','abcd','456');
insert into tb_user (id, nickname, username, password) values (3,'嘿嘿','aan','1234');
commit;
select * from tb_user;
```
jwt
----------------------------
```java
package com.example.demo.util;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;
import java.util.Date;
import java.util.UUID;
/**
 * JWT工具类
 */
public class JwtUtil {
    //有效期为
    public static final Long JWT_TTL = 60 * 60 *1000L;// 60 * 60 *1000  一个小时
    //设置秘钥明文
    public static final String JWT_KEY = "sangeng";
    public static String getUUID(){
        String token = UUID.randomUUID().toString().replaceAll("-", "");
        return token;
    }
    /**
     * 生成jtw
     * @param subject token中要存放的数据（json格式）
     * @return
     */
    public static String createJWT(String subject) {
        JwtBuilder builder = getJwtBuilder(subject, null, getUUID());// 设置过期时间
        return builder.compact();
    }
    /**
     * 生成jtw
     * @param subject token中要存放的数据（json格式）
     * @param ttlMillis token超时时间
     * @return
     */
    public static String createJWT(String subject, Long ttlMillis) {
        JwtBuilder builder = getJwtBuilder(subject, ttlMillis, getUUID());// 设置过期时间
        return builder.compact();
    }
    private static JwtBuilder getJwtBuilder(String subject, Long ttlMillis, String uuid) {
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
        SecretKey secretKey = generalKey();
        long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);
        if(ttlMillis==null){
            ttlMillis=JwtUtil.JWT_TTL;
        }
        long expMillis = nowMillis + ttlMillis;
        Date expDate = new Date(expMillis);
        return Jwts.builder()
                .setId(uuid)              //唯一的ID
                .setSubject(subject)   // 主题  可以是JSON数据
                .setIssuer("sg")     // 签发者
                .setIssuedAt(now)      // 签发时间
                .signWith(signatureAlgorithm, secretKey) //使用HS256对称加密算法签名, 第二个参数为秘钥
                .setExpiration(expDate);
    }
    /**
     * 创建token
     * @param id
     * @param subject
     * @param ttlMillis
     * @return
     */
    public static String createJWT(String id, String subject, Long ttlMillis) {
        JwtBuilder builder = getJwtBuilder(subject, ttlMillis, id);// 设置过期时间
        return builder.compact();
    }
    public static void main(String[] args) throws Exception {
        String token = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiJjYWM2ZDVhZi1mNjVlLTQ0MDAtYjcxMi0zYWEwOGIyOTIwYjQiLCJzdWIiOiJzZyIsImlzcyI6InNnIiwiaWF0IjoxNjM4MTA2NzEyLCJleHAiOjE2MzgxMTAzMTJ9.JVsSbkP94wuczb4QryQbAke3ysBDIL5ou8fWsbt_ebg";
        Claims claims = parseJWT(token);
        System.out.println(claims);
    }
    /**
     * 生成加密后的秘钥 secretKey
     * @return
     */
    public static SecretKey generalKey() {
        byte[] encodedKey = Base64.getDecoder().decode(JwtUtil.JWT_KEY);
        SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");
        return key;
    }
    /**
     * 解析
     *
     * @param jwt
     * @return
     * @throws Exception
     */
    public static Claims parseJWT(String jwt) throws Exception {
        SecretKey secretKey = generalKey();
        return Jwts.parser()
                .setSigningKey(secretKey)
                .parseClaimsJws(jwt)
                .getBody();
    }
}
```
## JsonData
```java
package com.example.demo.util;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
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
## R
```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.TypeReference;
import org.springframework.http.HttpStatus;
import java.util.HashMap;
import java.util.Map;
/**
 * @author:93879
 * @date:2023/06/09/23:31
 * @description:
 */
public class R extends HashMap<String, Object> {
    private static final long serialVersionUID = 1L;
    /**
     * 存元素 和setData的区别是 这个方法可以指定key的值
     */
    @Override
    public R put(String key, Object value) {
        super.put(key, value);
        return this;
    }
    public R setData(Object data) {
        put("data",data);
        return this;
    }
    /**
     * 获取数据
     * 远程调用服务时，另一个服务也会返回R类型的数据，这个方法可以获取key为data的value值
     * 利用fastjson进行反序列化
     */
    public <T> T getData(TypeReference<T> typeReference) {
        Object data = get("data");	//默认是map
        String jsonString = JSON.toJSONString(data);//转为json字符串
        T t = JSON.parseObject(jsonString, typeReference);//转为对象
        return t;
    }
    /**
     * 利用fastjson进行反序列化
     */
    public <T> T getData(String key,TypeReference<T> typeReference) {
        Object data = get(key);	//默认是map
        //转为json字符串
        String jsonString = JSON.toJSONString(data);//转为json字符串
        //转成需要的对象
        T t = JSON.parseObject(jsonString, typeReference);//转为对象
        return t;
    }
    public static long getSerialVersionUID() {
        return serialVersionUID;
    }
    //无参构造
    public R() {
        put("code", 200);
        put("msg", "success");
    }
    //返回错误码500的错误，错误内容：未知异常，请联系管理员
    public static R error() {
        return error(HttpStatus.INTERNAL_SERVER_ERROR.value(), "未知异常，请联系管理员");
    }
    //返回错误码500的错误，错误内容需要自己传入
    public static R error(String msg) {
        return error(HttpStatus.INTERNAL_SERVER_ERROR.value(), msg);
    }
    //返回自定义错误码，自定义错误内容
    public static R error(int code, String msg) {
        R r = new R();
        r.put("code", code);
        r.put("msg", msg);
        return r;
    }
    //返回正常，内容需要自己传入
    public static R ok(String msg) {
        R r = new R();
        r.put("msg", msg);
        return r;
    }
    //存入多条返回消息
    public static R ok(Map<String, Object> map) {
        R r = new R();
        r.putAll(map);
        return r;
    }
    //默认返回 0 success
    public static R ok() {
        return new R();
    }
    //获取code（状态码）的值
    public Integer getCode() {
        return (Integer) this.get("code");
    }
}
```
拦截器
----------------------------------------------------
```java
package com.example.demo.interceptor;
import com.example.demo.util.JwtUtil;
import io.jsonwebtoken.Claims;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.servlet.HandlerInterceptor;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * 自定义拦截器
 *
 * @author tong
 * @date 2023/03/13
 */
@Component
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 先获取请求头里的token
        String token = request.getHeader("token");
        // token判空
        if(StringUtils.isEmpty(token)){
            System.out.println("这次的请求是："+request.getServletPath());
            throw new RuntimeException("未登录，请登录后重试!");
        }
        // token解析是否成功，若成功，则放行；否则，不放行
        try {
            Claims claims = JwtUtil.parseJWT(token);
            String subject = claims.getSubject();
            System.out.println("sub:"+subject);
        } catch (Exception e) {
            throw new RuntimeException("token解析错误，请稍后重试!");
        }
        // 返回值代表是否放行
        return true;
    }
}
```
```java
package com.example.demo.config;
import com.example.demo.interceptor.MyInterceptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
/**
 * 拦截器配置
 *
 * @author tong
 * @date 2023/03/13
 */
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Autowired
    private MyInterceptor myInterceptor;
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 添加拦截器
        registry.addInterceptor(myInterceptor)
                // 添加拦截路径
                .addPathPatterns("/**")
                // 配置排除路径
                .excludePathPatterns("/user/login","/favicon.ico");
    }
}
```
异常处理
---------------------------------------------------------------
```java
package com.example.demo.handler;
import com.example.demo.util.JsonData;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
@RestControllerAdvice
public class AdviceController {
    @ExceptionHandler(RuntimeException.class)
    public Object handlerException(Exception e){
        // 获取异常信息，并返回
        return JsonData.buildError(e.getMessage());
    }
}
```
自定义参数解析
------------------------------------------------------------------------------------------------
> 许多的请求都需要获取到请求头里的token进行解析，在每个请求里都写解析的代码吗？
>
> 选用以下的自定义参数解析会好很多
```java
package com.example.demo.resolver;
import java.lang.annotation.*;
/**
 * 自定义注解
 *
 * @author tong
 * @date 2023/03/13
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.PARAMETER })
public @interface CurrentUserId {
}
```
```java
package com.example.demo.resolver;
import com.example.demo.util.JwtUtil;
import io.jsonwebtoken.Claims;
import org.springframework.core.MethodParameter;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;
/**
 * 用户id参数解析器
 *
 * @author tong
 * @date 2023/03/13
 */
@Component
public class UserIdArgumentResolver implements HandlerMethodArgumentResolver {
    /**
     * 是否能使用当前的参数解析器进行解析
     * @return 是否进行解析
     */
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        // 如果参数上有CurrentUserId注解，则可以被解析，如果不包含，则不能被解析
        return parameter.hasParameterAnnotation(CurrentUserId.class);
    }
    /**
     * 进行参数解析的方法，可以在方法中获取对应的数据，然后把数据作为返回值返回，方法的返回值就会赋值给对应的方法参数
     */
    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        // 获取请求头中的token
        String token = webRequest.getHeader("token");
        if(StringUtils.hasText(token)){
            // 解析token
            Claims claims = JwtUtil.parseJWT(token);
            return Integer.valueOf(claims.getSubject());
        }
        return null;
    }
}
```
```java
package com.example.demo.config;
import com.example.demo.resolver.UserIdArgumentResolver;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import java.util.List;
/**
 * 参数解析器配置
 *
 * @author tong
 * @date 2023/03/13
 */
@Configuration
public class ArgumentResolverConfig implements WebMvcConfigurer {
    @Autowired
    private UserIdArgumentResolver userIdArgumentResolver;
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(userIdArgumentResolver);
    }
}
```
> 测试
```java
@RestController
@RequestMapping("user")
public class UserController {
    @Autowired
    private UserService userService;
    @PostMapping("/login")
    public Object checkLogin(@RequestBody LoginVo loginVo){
        return userService.checkLogin(loginVo);
    }
    @PostMapping("/test1")
    public Object test1(@CurrentUserId Integer userId){
        return JsonData.buildSuccess(userId);
    }
}
```
![id解析](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152337575.png)
> 可以看到参数已经被解析成id了
跨域
-----------------------------------------
```java
package com.example.demo.config;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
/**
 * 跨域配置
 *
 * @author tong
 * @date 2023/03/13
 */
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        // 设置允许跨域的路径
        registry.addMapping("/**")
                // 设置允许跨域的域名
                .allowedOrigins("*")
                //是否允许cookie
                .allowCredentials(true)
                // 设置允许的请求方式
                .allowedMethods("GET","POST","DELETE","PUT")
                // 设置允许的header属性
                .allowedHeaders("*")
                // 跨域允许时间
                .maxAge(3600);
    }
}
```
参数校验
---------------------------------------------------------------
> oracle实现id自增和mysql有点区别
**新建分组**
```java
public interface AddGroup {
}
```
```java
public interface DeleteGroup {
}
```
```java
public interface UserMapper extends BaseMapper<User> {
    @Insert("INSERT INTO TB_USER (ID,NICKNAME,USERNAME,PASSWORD) VALUES ((SELECT MAX(ID)+1 FROM TB_USER),#{nickname},#{username},#{password})")
    boolean insertUser(User user);
}
```
```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User>
        implements UserService {
    @Override
    public Object checkLogin(LoginVo loginVo) {
        User user = baseMapper.selectOne(
                new QueryWrapper<User>()
                        .eq("USERNAME", loginVo.getUsername())
                        .eq("PASSWORD", loginVo.getPassword()));
        if(user==null){
            return JsonData.buildError("用户不存在");
        }else{
            return JsonData.buildSuccess(JwtUtil.createJWT(user.getId().toString()));
        }
    }
    @Override
    public boolean saveUser(User user) {
        return baseMapper.insertUser(user);
    }
}
```
```java
@RestController
@RequestMapping("user")
public class UserController {
    @Autowired
    private UserService userService;
    @PostMapping("/login")
    public Object checkLogin(@RequestBody LoginVo loginVo) {
        return userService.checkLogin(loginVo);
    }
    @PostMapping("/test1")
    public Object test1(@CurrentUserId Integer userId, @RequestBody User user) {
        HashMap<String, Object> map = new HashMap<>();
        map.put("id", userId);
        map.put("user", user);
        return JsonData.buildSuccess(map);
    }
    @PostMapping("/add")
    // 指定分组
    public Object addUser(@Validated({AddGroup.class}) @RequestBody User user) {
        if(userService.saveUser(user)){
            User one = userService.getOne(new QueryWrapper<User>()
                    .eq("USERNAME", user.getUsername())
                    .eq("PASSWORD", user.getPassword()));
            return JsonData.buildSuccess(one);
        }else{
            return JsonData.buildError("注册失败");
        }
    }
}
```
```java
@Data
@TableName(value ="TB_USER")
public class User implements Serializable {
    /**
     * id
     */
    @Null(message = "新增用户时，不能指定用户id",groups = {AddGroup.class})
    @NotNull(message = "删除用户时，必须指定用户id",groups = {DeleteGroup.class})
    @TableId(value = "ID",type = IdType.AUTO)
    private Integer id;
    /**
     * 昵称
     */
    @TableField(value = "NICKNAME")
    private String nickname;
    /**
     * 用户名
     */
    @TableField(value = "USERNAME")
    private String username;
    /**
     * 密码
     */
    @TableField(value = "PASSWORD")
    private String password;
    @TableField(exist = false)
    private static final long serialVersionUID = 1L;
}
```
 我们在日常开发中需要对程序内部的运行情况进行监控， 比如：健康度、运行指标、日志信息、线程状况等等 。而SpringBoot的监控Actuator就可以帮我们解决这些问题。
### 监控指标
①添加依赖
```xml
<dependency>
 	<groupId>org.springframework.boot</groupId>
 	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
②访问监控接口
[http://localhost:81/actuator](http://localhost:81/actuator)
③配置启用监控端点
```yaml
management:
  endpoints:
    enabled-by-default: true #配置启用所有端点
	web:
      exposure:
        include: "*" #web端暴露所有端点
```
### 常用端点
| 端点名称         | 描述                                      |
| :--------------- | :---------------------------------------- |
| `beans`          | 显示应用程序中所有Spring Bean的完整列表。 |
| `health`         | 显示应用程序运行状况信息。                |
| `info`           | 显示应用程序信息。                        |
| `loggers`        | 显示和修改应用程序中日志的配置。          |
| `metrics`        | 显示当前应用程序的“指标”信息。            |
| `mappings`       | 显示所有`@RequestMapping`路径列表。       |
| `scheduledtasks` | 显示应用程序中的计划任务。                |
### 图形化界面 SpringBoot Admin
①创建SpringBoot Admin Server应用
要求引入spring-boot-admin-starter-server依赖
```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
</dependency>
```
然后在启动类上加上@EnableAdminServer注解
②配置SpringBoot Admin client应用
在需要监控的应用中加上spring-boot-admin-starter-client依赖
```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>2.3.1</version>
</dependency>
```
然后配置SpringBoot Admin Server的地址
```yaml
spring:
  boot:
    admin:
      client:
        url: http://localhost:8888 #配置 Admin Server的地址
```
