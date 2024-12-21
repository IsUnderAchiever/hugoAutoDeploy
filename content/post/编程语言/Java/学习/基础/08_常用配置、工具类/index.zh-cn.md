---
title: 08_常用配置、工具类
description: 08_常用配置、工具类
date: 2023-03-05
slug: 08_常用配置、工具类
image: 202412212036798.png
categories:
    - Java
---

## 返回对象工具类
```java
import java.io.Serializable;
public class Result<T> implements Serializable {//返回的结果集类
    //服务器响应的状态   不是http状态
    private Integer code;
    //备注
    private String msg;
    //返回的数据
    private T data;
    public Result() {
    }
    public Result(Integer code, String msg, T data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }
    public Integer getCode() {
        return code;
    }
    public Result<?> setCode(Integer code) {
        this.code = code;
        return this;
    }
    public String getMsg() {
        return msg;
    }
    public Result<?> setMsg(String msg) {
        this.msg = msg;
        return this;
    }
    public T getData() {
        return data;
    }
    public Result<?> setData(T data) {
        this.data = data;
        return this;
    }
    @Override
    public String toString() {
        return "Result{" +
                "code=" + code +
                ", msg='" + msg + '\'' +
                ", data=" + data +
                '}';
    }
    public Result<?> setCode(ResultEnum resultEnum) {
        this.code = resultEnum.code;
        return this;
    }
}
```
```java
/**
 * 结果枚举
 *
 * @author tong
 * @date 2023/03/04
 */
public enum ResultEnum {
	/**
	 * 成功
	 */
	SUCCESS(200, "操作成功"),
	/**
	 * 对象创建成功
	 */
	CREATED(201, "对象创建成功"),
	/**
	 * 请求已经被接受
	 */
	ACCEPTED(202, "请求已经被接受"),
	/**
	 * 操作已经执行成功，但是没有返回数据
	 */
	NO_CONTENT(204, "操作已经执行成功，但是没有返回数据"),
	/**
	 * 资源已被移除
	 */
	MOVED_PERM(301, "资源已被移除"),
	/**
	 * 重定向
	 */
	SEE_OTHER(303, "重定向"),
	/**
	 * 资源没有被修改
	 */
	NOT_MODIFIED(304, "资源没有被修改"),
	/**
	 * 参数列表错误（缺少，格式不匹配）
	 */
	BAD_REQUEST(400, "操作失败"),
	/**
	 * 未授权
	 */
	UNAUTHORIZED(401, "未授权"),
	/**
	 * 访问受限，授权过期
	 */
	FORBIDDEN(403, "访问受限，授权过期"),
	/**
	 * 资源、服务未找到
	 */
	NOT_FOUND(404, "资源，服务未找到"),
	/**
	 * 不允许的http方法
	 */
	BAD_METHOD(405, "不允许的http方法"),
	/**
	 * 资源冲突，或者资源被锁
	 */
	CONFLICT(409, "资源冲突，或者资源被锁"),
	/**
	 * 不支持的数据、媒体类型
	 */
	UNSUPPORTED_TYPE(415, "不支持的数据、媒体类型"),
	/**
	 * 服务器内部错误
	 */
	ERROR(500, "服务器内部错误"),
	/**
	 * 接口未实现
	 */
	NOT_IMPLEMENTED(501, "接口未实现");
	public final Integer code;
	public final String msg;
	ResultEnum(Integer code, String msg) {
		this.code = code;
		this.msg = msg;
	}
	public Integer getCode() {
		return code;
	}
	public String getMsg() {
		return msg;
	}
}
```
```java
/**
 * 返回结果工具类
 *
 * @author tong
 * @date 2023/03/04
 */
public class ResultUtil {
    public static <T> Result<T>  defineSuccess(Integer code, T data) {
        Result result = new Result<>();
        return result.setCode(code).setData(data);
    }
    public static <T> Result<T> success(T data) {
        Result result = new Result();
        result.setCode(ResultEnum.SUCCESS).setData(data);
        return result;
    }
    public static <T> Result<T> fail(String msg) {
        Result result = new Result();
        result.setCode(ResultEnum.BAD_REQUEST).setMsg(msg);
        return result;
    }
    public static <T> Result<T> defineFail(Integer code, String msg){
        Result result = new Result();
        result.setCode(code).setMsg(msg);
        return result;
    }
    public static <T> Result<T> define(Integer code, String msg, T data){
        Result result = new Result();
        result.setCode(code).setMsg(msg).setData(data);
        return result;
    }
    public static Object defineSuccess(ResultEnum success, Object list) {
        Result result=new Result();
        result.setCode(success);
        result.setData(list);
        return result;
    }
}
```
> 用法
```java
@RestController
@RequestMapping("user")
public class UserController {
	@Autowired
	private UserService userService;
	@GetMapping("/id")
	public Object getUser() {
		return ResultUtil.define(
            ResultEnum.SUCCESS.getCode(), 
            ResultEnum.SUCCESS.getMsg(), 
            userService.list()
        );
	}
}
```
![image-20230305093542896](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303050935035.png)
> 其他返回工具类
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
```java
/**
 * @Auther: Administrator
 * @Date: 2022/10/12/9:09
 * @Description:
 */
import java.io.Serializable;
public class JsonResult<T> implements Serializable {
    private Boolean success;
    private Integer errorCode;
    private String errorMsg;
    private T data;
    public JsonResult() {
    }
    public JsonResult(boolean success) {
        this.success = success;
        this.errorCode = success ? com.example.demo.util.ResultCode.SUCCESS.getCode() : com.example.demo.util.ResultCode.COMMON_FAIL.getCode();
        this.errorMsg = success ? com.example.demo.util.ResultCode.SUCCESS.getMessage() : com.example.demo.util.ResultCode.COMMON_FAIL.getMessage();
    }
    public JsonResult(boolean success, com.example.demo.util.ResultCode resultEnum) {
        this.success = success;
        this.errorCode = success ? com.example.demo.util.ResultCode.SUCCESS.getCode() : (resultEnum == null ? com.example.demo.util.ResultCode.COMMON_FAIL.getCode() : resultEnum.getCode());
        this.errorMsg = success ? com.example.demo.util.ResultCode.SUCCESS.getMessage() : (resultEnum == null ? com.example.demo.util.ResultCode.COMMON_FAIL.getMessage() : resultEnum.getMessage());
    }
    public JsonResult(boolean success, T data) {
        this.success = success;
        this.errorCode = success ? com.example.demo.util.ResultCode.SUCCESS.getCode() : com.example.demo.util.ResultCode.COMMON_FAIL.getCode();
        this.errorMsg = success ? com.example.demo.util.ResultCode.SUCCESS.getMessage() : com.example.demo.util.ResultCode.COMMON_FAIL.getMessage();
        this.data = data;
    }
    public JsonResult(boolean success, com.example.demo.util.ResultCode resultEnum, T data) {
        this.success = success;
        this.errorCode = success ? com.example.demo.util.ResultCode.SUCCESS.getCode() : (resultEnum == null ? com.example.demo.util.ResultCode.COMMON_FAIL.getCode() : resultEnum.getCode());
        this.errorMsg = success ? com.example.demo.util.ResultCode.SUCCESS.getMessage() : (resultEnum == null ? com.example.demo.util.ResultCode.COMMON_FAIL.getMessage() : resultEnum.getMessage());
        this.data = data;
    }
    public Boolean getSuccess() {
        return success;
    }
    public void setSuccess(Boolean success) {
        this.success = success;
    }
    public Integer getErrorCode() {
        return errorCode;
    }
    public void setErrorCode(Integer errorCode) {
        this.errorCode = errorCode;
    }
    public String getErrorMsg() {
        return errorMsg;
    }
    public void setErrorMsg(String errorMsg) {
        this.errorMsg = errorMsg;
    }
    public T getData() {
        return data;
    }
    public void setData(T data) {
        this.data = data;
    }
}
```
```java
/**
 * @Auther: Administrator
 * @Date: 2022/10/12/9:09
 * @Description:
 */
public enum ResultCode {
    /* 成功 */
    SUCCESS(200, "成功"),
    /* 默认失败 */
    COMMON_FAIL(999, "失败"),
    /* 参数错误：1000～1999 */
    PARAM_NOT_VALID(1001, "参数无效"),
    PARAM_IS_BLANK(1002, "参数为空"),
    PARAM_TYPE_ERROR(1003, "参数类型错误"),
    PARAM_NOT_COMPLETE(1004, "参数缺失"),
    /* 用户错误 */
    USER_NOT_LOGIN(2001, "用户未登录"),
    USER_ACCOUNT_EXPIRED(2002, "账号已过期"),
    USER_CREDENTIALS_ERROR(2003, "密码错误"),
    USER_CREDENTIALS_EXPIRED(2004, "密码过期"),
    USER_ACCOUNT_DISABLE(2005, "账号不可用"),
    USER_ACCOUNT_LOCKED(2006, "账号被锁定"),
    USER_ACCOUNT_NOT_EXIST(2007, "账号不存在"),
    USER_ACCOUNT_ALREADY_EXIST(2008, "账号已存在"),
    USER_ACCOUNT_USE_BY_OTHERS(2009, "账号下线"),
    /* 业务错误 */
    NO_PERMISSION(3001, "没有权限");
    private Integer code;
    private String message;
    ResultCode(Integer code, String message) {
        this.code = code;
        this.message = message;
    }
    public Integer getCode() {
        return code;
    }
    public void setCode(Integer code) {
        this.code = code;
    }
    public String getMessage() {
        return message;
    }
    public void setMessage(String message) {
        this.message = message;
    }
    /**
     * 根据code获取message
     *
     * @param code
     * @return
     */
    public static String getMessageByCode(Integer code) {
        for (ResultCode ele : values()) {
            if (ele.getCode().equals(code)) {
                return ele.getMessage();
            }
        }
        return null;
    }
}
```
```java
/**
 * @Auther: Administrator
 * @Date: 2022/10/12/9:10
 * @Description:
 */
public class ResultTool {
    public static com.example.demo.util.JsonResult success() {
        return new com.example.demo.util.JsonResult(true);
    }
    public static <T> com.example.demo.util.JsonResult<T> success(T data) {
        return new com.example.demo.util.JsonResult(true, data);
    }
    public static com.example.demo.util.JsonResult fail() {
        return new com.example.demo.util.JsonResult(false);
    }
    public static com.example.demo.util.JsonResult fail(com.example.demo.util.ResultCode resultEnum) {
        return new com.example.demo.util.JsonResult(false, resultEnum);
    }
}
```
## Redis配置及工具类
### Redis配置
```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.serializer.SerializerFeature;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.type.TypeFactory;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.SerializationException;
import com.alibaba.fastjson.parser.ParserConfig;
import org.springframework.util.Assert;
import java.nio.charset.Charset;
/**
 * Redis使用FastJson序列化
 * 
 * @author sg
 */
public class FastJsonRedisSerializer<T> implements RedisSerializer<T>
{
    public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
    private Class<T> clazz;
    static
    {
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
    }
    public FastJsonRedisSerializer(Class<T> clazz)
    {
        super();
        this.clazz = clazz;
    }
    @Override
    public byte[] serialize(T t) throws SerializationException
    {
        if (t == null)
        {
            return new byte[0];
        }
        return JSON.toJSONString(t, SerializerFeature.WriteClassName).getBytes(DEFAULT_CHARSET);
    }
    @Override
    public T deserialize(byte[] bytes) throws SerializationException
    {
        if (bytes == null || bytes.length <= 0)
        {
            return null;
        }
        String str = new String(bytes, DEFAULT_CHARSET);
        return JSON.parseObject(str, clazz);
    }
    protected JavaType getJavaType(Class<?> clazz)
    {
        return TypeFactory.defaultInstance().constructType(clazz);
    }
}
```
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;
@Configuration
public class RedisConfig {
    @Bean
    @SuppressWarnings(value = { "unchecked", "rawtypes" })
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory connectionFactory)
    {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        FastJsonRedisSerializer serializer = new FastJsonRedisSerializer(Object.class);
        // 使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(serializer);
        // Hash的key也采用StringRedisSerializer的序列化方式
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);
        template.afterPropertiesSet();
        return template;
    }
}
```
### Redis工具类
```java
import java.util.*;
import java.util.concurrent.TimeUnit;
@SuppressWarnings(value = { "unchecked", "rawtypes" })
@Component
public class RedisCache
{
    @Autowired
    public RedisTemplate redisTemplate;
    /**
     * 缓存基本的对象，Integer、String、实体类等
     *
     * @param key 缓存的键值
     * @param value 缓存的值
     */
    public <T> void setCacheObject(final String key, final T value)
    {
        redisTemplate.opsForValue().set(key, value);
    }
    /**
     * 缓存基本的对象，Integer、String、实体类等
     *
     * @param key 缓存的键值
     * @param value 缓存的值
     * @param timeout 时间
     * @param timeUnit 时间颗粒度
     */
    public <T> void setCacheObject(final String key, final T value, final Integer timeout, final TimeUnit timeUnit)
    {
        redisTemplate.opsForValue().set(key, value, timeout, timeUnit);
    }
    /**
     * 设置有效时间
     *
     * @param key Redis键
     * @param timeout 超时时间
     * @return true=设置成功；false=设置失败
     */
    public boolean expire(final String key, final long timeout)
    {
        return expire(key, timeout, TimeUnit.SECONDS);
    }
    /**
     * 设置有效时间
     *
     * @param key Redis键
     * @param timeout 超时时间
     * @param unit 时间单位
     * @return true=设置成功；false=设置失败
     */
    public boolean expire(final String key, final long timeout, final TimeUnit unit)
    {
        return redisTemplate.expire(key, timeout, unit);
    }
    /**
     * 获得缓存的基本对象。
     *
     * @param key 缓存键值
     * @return 缓存键值对应的数据
     */
    public <T> T getCacheObject(final String key)
    {
        ValueOperations<String, T> operation = redisTemplate.opsForValue();
        return operation.get(key);
    }
    /**
     * 删除单个对象
     *
     * @param key
     */
    public boolean deleteObject(final String key)
    {
        return redisTemplate.delete(key);
    }
    /**
     * 删除集合对象
     *
     * @param collection 多个对象
     * @return
     */
    public long deleteObject(final Collection collection)
    {
        return redisTemplate.delete(collection);
    }
    /**
     * 缓存List数据
     *
     * @param key 缓存的键值
     * @param dataList 待缓存的List数据
     * @return 缓存的对象
     */
    public <T> long setCacheList(final String key, final List<T> dataList)
    {
        Long count = redisTemplate.opsForList().rightPushAll(key, dataList);
        return count == null ? 0 : count;
    }
    /**
     * 获得缓存的list对象
     *
     * @param key 缓存的键值
     * @return 缓存键值对应的数据
     */
    public <T> List<T> getCacheList(final String key)
    {
        return redisTemplate.opsForList().range(key, 0, -1);
    }
    /**
     * 缓存Set
     *
     * @param key 缓存键值
     * @param dataSet 缓存的数据
     * @return 缓存数据的对象
     */
    public <T> BoundSetOperations<String, T> setCacheSet(final String key, final Set<T> dataSet)
    {
        BoundSetOperations<String, T> setOperation = redisTemplate.boundSetOps(key);
        Iterator<T> it = dataSet.iterator();
        while (it.hasNext())
        {
            setOperation.add(it.next());
        }
        return setOperation;
    }
    /**
     * 获得缓存的set
     *
     * @param key
     * @return
     */
    public <T> Set<T> getCacheSet(final String key)
    {
        return redisTemplate.opsForSet().members(key);
    }
    /**
     * 缓存Map
     *
     * @param key
     * @param dataMap
     */
    public <T> void setCacheMap(final String key, final Map<String, T> dataMap)
    {
        if (dataMap != null) {
            redisTemplate.opsForHash().putAll(key, dataMap);
        }
    }
    /**
     * 获得缓存的Map
     *
     * @param key
     * @return
     */
    public <T> Map<String, T> getCacheMap(final String key)
    {
        return redisTemplate.opsForHash().entries(key);
    }
    /**
     * 往Hash中存入数据
     *
     * @param key Redis键
     * @param hKey Hash键
     * @param value 值
     */
    public <T> void setCacheMapValue(final String key, final String hKey, final T value)
    {
        redisTemplate.opsForHash().put(key, hKey, value);
    }
    /**
     * 获取Hash中的数据
     *
     * @param key Redis键
     * @param hKey Hash键
     * @return Hash中的对象
     */
    public <T> T getCacheMapValue(final String key, final String hKey)
    {
        HashOperations<String, String, T> opsForHash = redisTemplate.opsForHash();
        return opsForHash.get(key, hKey);
    }
    /**
     * 删除Hash中的数据
     * 
     * @param key
     * @param hkey
     */
    public void delCacheMapValue(final String key, final String hkey)
    {
        HashOperations hashOperations = redisTemplate.opsForHash();
        hashOperations.delete(key, hkey);
    }
    /**
     * 获取多个Hash中的数据
     *
     * @param key Redis键
     * @param hKeys Hash键集合
     * @return Hash对象集合
     */
    public <T> List<T> getMultiCacheMapValue(final String key, final Collection<Object> hKeys)
    {
        return redisTemplate.opsForHash().multiGet(key, hKeys);
    }
    /**
     * 获得缓存的基本对象列表
     *
     * @param pattern 字符串前缀
     * @return 对象列表
     */
    public Collection<String> keys(final String pattern)
    {
        return redisTemplate.keys(pattern);
    }
}
```
```java
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
public class WebUtils
{
    /**
     * 将字符串渲染到客户端
     * 
     * @param response 渲染对象
     * @param string 待渲染的字符串
     * @return null
     */
    public static String renderString(HttpServletResponse response, String string) {
        try
        {
            response.setStatus(200);
            response.setContentType("application/json");
            response.setCharacterEncoding("utf-8");
            response.getWriter().print(string);
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }
        return null;
    }
}
```
## JWT工具类
```java
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
## 密码加密存储
​	我们一般使用SpringSecurity为我们提供的BCryptPasswordEncoder。
​	我们只需要使用把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验。
​	我们可以定义一个SpringSecurity的配置类，SpringSecurity要求这个配置类要继承WebSecurityConfigurerAdapter。
~~~~java
/**
 * @Author 三更  B站： https://space.bilibili.com/663528522
 */
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
~~~~
## 加密解密工具类
```java
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.security.SecureRandom;
// 加密、解密工具类
public class CryptUtil {
    private static final String AES = "AES";
    private static int keysizeAES = 128;
    private static String charset = "UTF-8";
    public static String parseByte2HexStr(final byte[] b) {
        final StringBuilder stringBuffer = new StringBuilder();
        for (byte value : b) {
            String hex = Integer.toHexString(value & 0xFF);
            if (hex.length() == 1) {
                hex = '0' + hex;
            }
            stringBuffer.append(hex.toUpperCase());
        }
        return stringBuffer.toString();
    }
    public static byte[] parseHexStr2Byte(final String hexStr) {
        if (hexStr.length() < 1) {
            return null;
        }
        final byte[] result = new byte[hexStr.length() / 2];
        for (int i = 0; i < hexStr.length() / 2; i++) {
            int high = Integer.parseInt(hexStr.substring(i * 2, i * 2 + 1), 16);
            int low = Integer.parseInt(hexStr.substring(i * 2 + 1, i * 2 + 2), 16);
            result[i] = (byte) (high * 16 + low);
        }
        return result;
    }
    private static String keyGeneratorES(final String res, final String algorithm, final String key, final Integer keysize, final boolean bEncode) {
        try {
            final KeyGenerator g = KeyGenerator.getInstance(algorithm);
            if (keysize == 0) {
                byte[] keyBytes = charset == null ? key.getBytes() : key.getBytes(charset);
                g.init(new SecureRandom(keyBytes));
            } else if (key == null) {
                g.init(keysize);
            } else {
                byte[] keyBytes = charset == null ? key.getBytes() : key.getBytes(charset);
                SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
                random.setSeed(keyBytes);
                g.init(keysize, random);
            }
            final SecretKey secretKey = g.generateKey();
            final SecretKeySpec keySpec = new SecretKeySpec(secretKey.getEncoded(), algorithm);
            final Cipher cipher = Cipher.getInstance(algorithm);
            if (bEncode) {
                cipher.init(Cipher.ENCRYPT_MODE, keySpec);
                final byte[] result = charset == null ? res.getBytes() : res.getBytes(charset);
                return parseByte2HexStr(cipher.doFinal(result));
            } else {
                cipher.init(Cipher.DECRYPT_MODE, keySpec);
                return new String(cipher.doFinal(parseHexStr2Byte(res)));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
    public static String AESencode(final String res) {
        return keyGeneratorES(res, AES, "aAll*-%", keysizeAES, true);
    }
    public static String AESdecode(final String res) {
        return keyGeneratorES(res, AES, "aAll*-%", keysizeAES, false);
    }
    public static void main(String[] args) {
        System.out.println("加密后:" + AESencode("123456"));
        System.out.println("解密后:" + AESdecode("555B695B57E024EEA169EAE275B9D93B"));
    }
}
```
## 常用pom依赖
```xml
<!--Swagger3.x-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
<!--MyBatis-plus自动生成-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.1</version>
</dependency>
<!--velocity-->
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.0</version>
</dependency>
<!--代码自动生成依赖-->
<!--Json-->
<dependency>
    <groupId>com.vaadin.external.google</groupId>
    <artifactId>android-json</artifactId>
    <version>0.0.20131108.vaadin1</version>
    <scope>compile</scope>
</dependency>
<!--Swagger-->
<dependency>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-annotations</artifactId>
    <version>1.5.22</version>
</dependency>
```
## swagger配置
> 访问`http://localhost:8090/swagger-ui/index.html`即可查看接口文档
```xml
<!--Swagger3.x-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```
```properties
# ==========自定义swagger配置==========
swagger.enable=true
swagger.application-name=${spring.application.name}
swagger.application-version=1.0
swagger.application-description=1024shop api info
#swagger.application-description=1024shop电商平台管理后端接口文档
# 这里如果是中文，则会出现乱码，有以下两种解决方法
# 1. 修改文件编码
# 2. 使用yml格式
```
![image-20230305172841417](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303051728479.png)
![image-20230305172859280](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303051728336.png)
> 编码格式改为了`ISO-8859-1`，如果是配置yml文件，格式也需要更改
![image-20230305172922513](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303051729582.png)
![@PropertySource读取配置文件](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303051729342.png)
```java
import io.swagger.annotations.ApiOperation;
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.oas.annotations.EnableOpenApi;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
/**
 * @Auther: 温豪
 * @Date: 2022/11/30/15:05
 * @Description:
 */
@Component
@Data
@ConfigurationProperties("swagger") // 映射到properties配置，省略前缀
@EnableOpenApi // 开启规范
public class SwaggerConfiguration {
    /**
     * 是否开启swagger，生产环境一般关闭，所以这里定义一个变量
     */
    private Boolean enable;
    /**
     * 项目应用名
     */
    private String applicationName;
    /**
     * 项目版本信息
     */
    private String applicationVersion;
    /**
     * 项目描述信息
     */
    private String applicationDescription;
    @Bean
    public Docket docket() {
        return new Docket(DocumentationType.OAS_30)
                .pathMapping("/")
                // 定义是否开启swagger，false为关闭，可以通过变量控制，线上关闭
                .enable(enable)
                //配置api文档元信息
                .apiInfo(apiInfo())
                // 选择哪些接口作为swagger的doc发布
                .select()
                //apis() 控制哪些接口暴露给swagger，
                // RequestHandlerSelectors.any() 所有都暴露
                // RequestHandlerSelectors.basePackage("net.xdclass.*")  指定包位置
                // withMethodAnnotation(ApiOperation.class)标记有这个注解 ApiOperation
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title(applicationName)
                .description(applicationDescription)
                .contact(new Contact("深海火锅店", "tongstyle.gitee.io", "938798576@aliyun.com"))
                .version(applicationVersion)
                .build();
    }
}
```
```java
// 主要注解
// 1.@Api()
// 2.@ApiOperation()
// 3.@ApiResponse()
// 4.@ApiModel()
// 5.@ApiModelProperty()
@RestController
@RequestMapping("/user")
@Api(tags = "用户模块", value = "用户controller")
// --------------------------------------------------
@ApiOperation("用户登录")
@ApiResponses({
  @ApiResponse(responseCode = "404",description = "找不到该页面"),
  @ApiResponse(responseCode = "403",description = "权限不够")
})
@ApiImplicitParams({
			@ApiImplicitParam(name = "mobile", value = "手机号码", dataType = "string", paramType = "query", example = "13802780104", required = true),
			@ApiImplicitParam(name = "user_name", value = "登录账号", dataType = "string", paramType = "query", example = "lihailin9073", required = true), })
@PostMapping("/login")
public Object login(@RequestBody User user) {
  return userService.login(user);
}
// --------------------------------------------------
// 这里千万要给get、set方法或者Data注解
@Data
@ApiModel(value = "User对象", description = "用户模型")
public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    @TableId(value = "id", type = IdType.AUTO)
    @ApiModelProperty(value = "主键")
    private Integer id;
    @TableField("username")
    @ApiModelProperty(value = "用户名", required = true, example = "admin")
    private String username;
    @TableField("password")
    @ApiModelProperty(value = "密码", required = true, example = "123456")
    private String password;
}
```
> 不加`@RequestBody`的情景，可以直接在`Parameters`获取到
![image-20230305171247415](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303051712583.png)
> 加了`@RequestBody`的情景，显示`No parameters`，但`Schema`处可看到参数详情
![image-20230305171415219](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303051714339.png)
![image-20230305171448855](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303051714998.png)
## MyBatis-plus配置
```xml
<!-- fastjson -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.72</version>
</dependency>
<!--Swagger3.x-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
<!--新版 mybatis-plus 自动生成器-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.2</version>
</dependency>
<!-- freemarker -->
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.31</version>
</dependency>
<!-- web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- mybatis-plus -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.2</version>
</dependency>
<!-- mysql -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<!-- lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<!-- junit -->
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
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
</dependency>
```
### 自动生成
```java
import java.util.Collections;
import com.baomidou.mybatisplus.annotation.FieldFill;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.generator.FastAutoGenerator;
import com.baomidou.mybatisplus.generator.config.OutputFile;
import com.baomidou.mybatisplus.generator.config.rules.DateType;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;
public class GeneratorTest {
	public static void main(String[] args) {
		// 配置相关的数据库连接
		// TODO TODO
		FastAutoGenerator.create("jdbc:mysql://localhost:3306/test?serverTimezone=Asia/Shanghai", "root", "123456")
				.globalConfig(builder -> {
					builder.author("深海火锅店") // 设置作者
							.enableSwagger() // 开启 swagger 模式
							.fileOverride() // 覆盖已生成文件
							// TODO TODO
							.outputDir("src\\main\\java\\") // 指定输出目录
							.dateType(DateType.ONLY_DATE);
				}).packageConfig(builder -> {
					// TODO TODO
					builder.parent("com.example.demo") // 设置父包名
//                            .moduleName("") // 设置父包模块名
							.entity("domain").controller("controller").mapper("mapper").service("service")
							.serviceImpl("service.impl")
							.pathInfo(Collections.singletonMap(OutputFile.xml, "src\\main\\resources\\mapper\\")); // 设置mapperXml生成路径
				}).strategyConfig(builder -> {
					// TODO TODO
					builder.addInclude("user")// 设置需要生成的表名
					// TODO TODO
//                            .addTablePrefix("tb_") // 设置过滤表前缀
							.mapperBuilder()
							// 控制器controller配置
							.controllerBuilder()
							// 开启生成@RestController 控制器
							.enableRestStyle()
							// 开启驼峰转连字符
							.enableHyphenStyle()
					// 开启父类
//							.superClass("com.example.demo.controller.BaseController")
							// 控制器统一后缀
							.formatFileName("%sController")
							// service配置
							.serviceBuilder()
							// service统一后缀
							.formatServiceFileName("%sService")
							// serviceImpl统一后缀
							.formatServiceImplFileName("%sServiceImpl")
							// mapper配置
							.mapperBuilder()
							// 生成基础字段的map映射map
							.enableBaseResultMap()
							// 生成基础查询的sql
							.enableBaseColumnList()
							// 实体类配置
							.entityBuilder()
					// 开启链式模型，开启lombok模型不需要开启这个
//                            .enableChainModel()
							// 开启lombok模型
							.enableLombok()
							// 开启表字段注解
							.enableTableFieldAnnotation()
							// 逻辑删除字段
							// TODO TODO
							.logicDeleteColumnName("is_deleted")
							// 数据库表映射到实体的命名策略
							.naming(NamingStrategy.underline_to_camel)
					// 数据库表字段映射到实体的命名策略,驼峰命名，这个未设置会按照naming来配置
//                            .columnNaming(NamingStrategy.underline_to_camel)
					// 指定实体类父类
//                            .superClass("com.mybatisplus.generator.entity.BaseEntity")
					// 父类字段
//                            .addSuperEntityColumns("id", "create_id", "modify_id", "create_date", "modify_date", "is_deleted")
//                             开启 ActiveRecord 模式，即实体类继承Model类,自己提供CRUD操作,不建议使用,会和父类形成单继承冲突
//                            .enableActiveRecord()
							// 表字段填充字段，对应数据库字段，插入的时候自动填充
							.addTableFills(new Column("create_date", FieldFill.INSERT))
							// 表字段填充字段，对应实体类字段，插入的时候自动填充
							.addTableFills(new Property("createDate", FieldFill.INSERT))
							// 表字段填充字段，对应数据库字段，更新的时候自动填充
							.addTableFills(new Column("modify_date", FieldFill.UPDATE))
							// 表字段填充字段，对应实体类字段，更新的时候自动填充
							.addTableFills(new Property("modifyDate", FieldFill.UPDATE))
					// 忽略的字段
//							.addIgnoreColumns("version")
							// id自增
							.idType(IdType.AUTO);
					// 实体类统一后缀
				}).templateEngine(new FreemarkerTemplateEngine()) // 使用Freemarker引擎模板，默认的是Velocity引擎模板
				.execute();
	}
}
```
```properties
# 应用名称 TODO
spring.application.name=demo
# 应用服务 WEB 访问端口
server.port=8080
# 数据库驱动：
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 数据源名称
spring.datasource.name=defaultDataSource
# 数据库连接地址 TODO
spring.datasource.url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC
# 数据库用户名&密码： TODO
spring.datasource.username=root
spring.datasource.password=123456
# 配置mybatis-plus 打印sql日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
# xml文件路径
mybatis-plus.mapper-locations=classpath:/mapper/**/*.xml
# 配置最新全局配置文件
#mybatis-plus.config-location=classpath:mybatis-config.xml
# 配置mybatis-plus 包路径 TODO
mybatis-plus.type-aliases-package=com.example.demo.domain
# mybatis-plus下划线转驼峰配置，默认为true
mybatis-plus.configuration.map-underscore-to-camel-case=true
# 配置全局默认主键类型，实体类不用加@TableId(value ="id",type = IdType.AUTO)
mybatis-plus.global-config.db-config.id-type=auto
# 逻辑删除 （1为删除，0为未删除）
#mybatis-plus.global-config.db-config.logic-delete-value=1
#mybatis-plus.global-config.db-config.logic-not-delete-value=0
# 如果java实体类没加注解@TableLogic，则可以配置这个，推介这里配置
#mybatis-plus.global-config.db-config.logic-delete-field=is_deleted
```
```java
// 配置分页插件
@Configuration
public class MyBatisPlusPaginationInnerConfig {
    /**
     * 分页插件（官网最新）
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```
```java
// 分页测试
@Test
public void test1(){
    // QueryWrapper<User> wrapper = new QueryWrapper<>();
    // wrapper.eq("weight",4);
    //第1页，每页2条
    Page<User> page = new Page<>(1, 2);
    IPage<User> UserIPage = userMapper.selectPage(page, null);
    System.out.println("总条数"+UserIPage.getTotal());
    System.out.println("总页数"+UserIPage.getPages());
    //获取当前数据
    System.out.println(UserIPage.getRecords().toString());
}
```
## FastJson
[fastjson jar包](https://www.123pan.com/s/tMU0Vv-fFiUd.html)
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.72</version>
</dependency>
```
> 用法
```java
@Test
public void returnJson() {
    User user = new User(1, "admin", "123456", "哈哈");
    System.out.println("user1:" + user);
    // 对象转String
    System.out.println("user2:" + JSON.toJSONString(user));
    // 集合转json
    User user1 = new User(2, "张三", "123456", "呵呵");
    User user2 = new User(2, "李四", "000", "呵呵");
    List<User> users = new ArrayList<User>();
    users.add(user1);
    users.add(user2);
    System.out.println("集合:" + JSON.toJSONString(users));
    // 解析json对象
    System.out.println("解析:" + JSON.parseObject(JSON.toJSONString(user)));
}
```
> 结果
```java
user1:com.example.demo.domain.User@5b251fb9
user2:{"id":1,"nickName":"哈哈","password":"123456","username":"admin"}
集合:[{"id":2,"nickName":"呵呵","password":"123456","username":"张三"},{"id":2,"nickName":"呵呵","password":"000","username":"李四"}]
解析:{"password":"123456","nickName":"哈哈","id":1,"username":"admin"}
```
