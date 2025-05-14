---
title: 表单校验+异常处理
description: 表单校验+异常处理
date: 2023-02-18
slug: 表单校验+异常处理
image: 202412212133331.png
categories:
    - Spring
---

## 表单校验
`JSR303`
```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```
1. 给Bean添加校验注解
   如`@Email` 标注的字段为邮箱、`@NotNull` 标注的字段不能为空、`@Future` 标注的字段必须是未来的时间、`@Min`标注最小值
   ```java
   // 若不写message，则返回默认信息
   @NotBlank(message = "用户名不能为空")
   @TableField(value = "username")
   private String username;
   ```
   
2. 开启校验注解`@Valid`，告知spring方法需要开启校验这些字段，上面只是标注了校验规则，标注`（@Valid）`
   ```java
   @PostMapping("/test")
   public String test(@Valid @RequestBody User user){
       return "error1";
   }
   ```
3. 获取校验的结果，校验是否成功...，紧跟参数后面添加`BindingResult result`，示例如下：
   ```java
   @PostMapping("/test")
   public Object test(@Valid @RequestBody User user, BindingResult result){
       Map<String, Object> map = new HashMap<>();
       // 如果校验出错
       if (result.hasErrors()){
           Map<String, String> error = new HashMap<>();
           // 获取校验的错误结果
               result.getFieldErrors().forEach((item)->{
           // FieldError 获取到错误提示
           String message = item.getDefaultMessage();
           // 获取错误的属性名字
           String field = item.getField();
           error.put(field,message);
           });
           map.put("code",500);
           map.put("message",error);
       }else{
           // 校验未出错
           map.put("code",200);
           map.put("message",user);
       }
       return map;
   }
   ```
   ![image-20230218140813717](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181408949.png)
   
   ![image-20230218140908480](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181409557.png)
   
### 自定义校验规则
```java
// regexp内写 正则表达式    
@Pattern(regexp = "/^[a-zA-Z0-9]$/",message = "昵称必须是一个字母或数字")
@TableField(value = "nick_name")
private String nick_name;
```
## 配置全局异常处理
> 1. 类添加注解
>
>    - @ControllerAdvice 如果需要返回json数据，则需要在方法上加上@ResponseBody
>
>    - @RestControllerAdvice，默认返回json数据，方法不需要加@ResponseBody
>
>      `区别`:参考@RestController和@Controller的
>      
> 2. 方法添加处理器
>      
>    - 捕获全局异常，处理所有不可知异常
>
>    - @ExceptionHandler(value=Exception.class)
```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
@Slf4j
@RestControllerAdvice(basePackages = "com.example.exceptiontest.controller")
public class UserExceptionControllerAdvice {
    @ExceptionHandler(value = Exception.class)
    public Object handleValidException(Exception e){
        HashMap<String, String> map = new HashMap<>();
        map.put("message",e.getMessage());
        map.put("class",e.getClass().toString());
        log.error("数据校验出现异常:{}，异常类型:{}",e.getMessage(),e.getClass());
        return map;
    }
}
```
既然要进行异常处理，那么上方配置的校验就需要抛出异常，去掉BindingResult即可
```java
@PostMapping("/test2")
public Object test2(@Valid @RequestBody User user){
    Map<String, Object> map = new HashMap<>();
    map.put("code",200);
    map.put("message",user);
    return map;
}
```
![image-20230218143001936](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181430162.png)
![image-20230218143053836](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181430912.png)
> 指定的异常类型需要更精确才行
![image-20230218143237871](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181432950.png)
```java
@Slf4j
@RestControllerAdvice(basePackages = "com.example.exceptiontest.controller")
public class UserExceptionControllerAdvice {
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    public Object handleValidException(MethodArgumentNotValidException e){
        HashMap<String, String> map = new HashMap<>();
        BindingResult bindingResult = e.getBindingResult();
        map.put("code","400");
        bindingResult.getFieldErrors().forEach((item)->{
            map.put(item.getField(),item.getDefaultMessage());
        });
        map.put("msg","数据校验出现异常");
        log.error("数据校验出现异常:{}，异常类型:{}",e.getMessage(),e.getClass());
        return map;
    }
}
```
![image-20230218143612705](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181436807.png)
> 为了更加统一、方便地编写返回状态码，可创建一个`枚举类`
```java
public enum BizCodeEnume {
    /**
     * 系统未知异常
     */
    UNKNOWN_EXCEPTION(10000,"系统未知异常"),
    /**
     * 参数格式校验失败
     */
    VAILD_EXCEPTION(10001,"参数格式校验失败");
    private final int code;
    private final String msg;
    BizCodeEnume(int code, String msg){
        this.code = code;
        this.msg = msg;
    }
    /**
     * 获取状态码
     *
     * @return int
     */
    public int getCode() {
        return code;
    }
    /**
     * 获取信息
     *
     * @return {@link String}
     */
    public String getMsg() {
        return msg;
    }
}
```
```java
@Slf4j
@RestControllerAdvice(basePackages = "com.example.exceptiontest.controller")
public class UserExceptionControllerAdvice {
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    public Object handleValidException(MethodArgumentNotValidException e){
        HashMap<String, String> map = new HashMap<>();
        BindingResult bindingResult = e.getBindingResult();
        map.put("code", String.valueOf(BizCodeEnume.VAILD_EXCEPTION.getCode()));
        bindingResult.getFieldErrors().forEach((item)->{
            map.put(item.getField(),item.getDefaultMessage());
        });
        map.put("msg",BizCodeEnume.VAILD_EXCEPTION.getMsg());
        log.error("数据校验出现异常:{}，异常类型:{}",e.getMessage(),e.getClass());
        return map;
    }
    /**
     * 处理最大的异常
     *
     * @param e e
     * @return {@link Object}
     */
    @ExceptionHandler(value = Throwable.class)
    public Object handleException(Throwable e){
        HashMap<String, String> map = new HashMap<>();
        map.put("code",String.valueOf(BizCodeEnume.UNKNOWN_EXCEPTION.getCode()));
        map.put("msg",BizCodeEnume.UNKNOWN_EXCEPTION.getMsg());
        return map;
    }
}
```
![image-20230218144806627](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181448835.png)
### 自定义异常
> 注释掉之前写过的`@NotBlank(message = "用户名不能为空")`...参数校验注解
```java
public class UsernameOrPasswordNULLException extends RuntimeException {
    public UsernameOrPasswordNULLException(String message) {
        super(message);
    }
}
```
```java
// 捕获UsernameOrPasswordNULLException异常
@ExceptionHandler(value = UsernameOrPasswordNULLException.class)
public Object usernameOrPasswordIsNull(UsernameOrPasswordNULLException e){
    HashMap<String, String> map = new HashMap<>();
    // 获取状态码
    map.put("code", "501");
    // 获取信息
    map.put("msg","用户名或密码为空");
    return map;
}
```
```java
@PostMapping("/test3")
public Object test3(@RequestBody User user){
    String username=user.getUsername();
    String password=user.getPassword();
    if(username==null || "".equals(username)||password==null ||"".equals(password)){
        throw new UsernameOrPasswordNULLException("用户名或密码不能为空");
    }
    Map<String, Object> map = new HashMap<>();
    map.put("code",200);
    map.put("message",user);
    return map;
}
```
![image-20230218150809676](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181508917.png)
## 分组校验
> 各个业务场景下的参数校验规则可能是不一样的
> 比如`新增用户`时，id字段自增，所以不需要填写，但`删除用户`时，id字段必须要填写
>
> 此时需要使用到`分组校验功能`
给注解标明，何时使用A校验规则，何时使用B校验规则
```java
// 指定组AddGroup，新建AddGroup接口，什么都不用写;DeleteGroup 也一样
@Null(message = "新增用户时，不能指定用户id",groups = {AddGroup.class})
@NotNull(message = "删除用户时，必须指定用户id",groups = {DeleteGroup.class})
@TableId(value = "id", type = IdType.AUTO)
private Integer id;
```
```java
public interface AddGroup {
}
public interface DeleteGroup {
}
```
![image-20230218152344005](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181523120.png)
> 将`@Valid`注解 改为 `@Validated`注解
```java
    @PostMapping("/save")
    public Object saveUser(@Validated({AddGroup.class}) @RequestBody User user) {
        boolean saveFlag = userService.save(user);
        Map<String, Object> map = new HashMap<>();
        map.put("code", 200);
        map.put("message", saveFlag ? "保存成功" : "保存失败");
        return map;
    }
    @DeleteMapping("/delete")
    public Object deleteUser(@Validated({DeleteGroup.class}) @RequestBody User user) {
        boolean saveFlag = userService.removeById(user.getId());
        Map<String, Object> map = new HashMap<>();
        map.put("code", 200);
        map.put("message", saveFlag ? "删除成功" : "删除失败");
        return map;
    }
```
**若校验注解如@NotNull不指定group，那么在执行方法时，会校验吗？**
答案是不会校验，不标注分组的校验注解是不起作用的
![image-20230218153455850](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181534110.png)
![image-20230218153544904](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181535976.png)
![image-20230218153625006](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181536074.png)
![image-20230218153647928](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181536000.png)
## 自定义校验
> 若`正则表达式`无法满足业务需求，需要自定义校验逻辑
1. 编写自定义校验注解
2. 编写自定义校验器
3. 关联校验器和注解，让校验器来校验注解标注的字段
```java
@NickValue(vals={"喜羊羊","灰太狼"})
@TableField(value = "nick_name")
private String nick_name;
```
```java
import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.*;
@Documented
// 这里需要指定校验器，暂时先空着，等到后面再编写校验器
@Constraint(
        validatedBy = {}
)
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE, ElementType.CONSTRUCTOR, ElementType.PARAMETER, ElementType.TYPE_USE})
@Retention(RetentionPolicy.RUNTIME)
public @interface NickValue {
    /**
     * 消息
     * 这里写该注解的全类名
     * 若校验失败，会前往ValidationMessages.properties来查询失败信息
     * 所以我们需要自己配置一个ValidationMessages.properties
     *
     * @return {@link String}
     */
    String message() default "{com.example.exceptiontest.valid.NickValue.message}";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    String[] vals() default {};
}
```
![image-20230218155417292](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181554598.png)
```properties
com.example.exceptiontest.valid.NickValue.message=昵称必须为指定的值
```
**编写自定义校验器**
```java
import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
import java.util.HashSet;
import java.util.Set;
/**
 * 自定义昵称校验器
 * ConstraintValidator<NickValue,String> 前面为注解，后面为校验的字段类型
 * 如此时校验的字段是nickname，它的类型是String
 *
 * @author tong
 * @date 2023/02/18
 */
public class NickValueConstraintValidator implements ConstraintValidator<NickValue,String> {
    private Set<String> set=new HashSet<String>();
    /**
     * 初始化
     *
     * @param constraintAnnotation 约束注释
     */
    @Override
    public void initialize(NickValue constraintAnnotation) {
        String[] vals = constraintAnnotation.vals();
        for (String val : vals) {
            set.add(val);
        }
    }
    /**
     * 是否校验成功
     *
     * @param s                          昵称
     * @param constraintValidatorContext 约束验证器上下文
     * @return boolean
     */
    @Override
    public boolean isValid(String s, ConstraintValidatorContext constraintValidatorContext) {
        if(set.contains(s)){
            return true;
        }
        return false;
    }
}
```
> 别忘了在注解上添加对应的校验器
```java
@Documented
// 添加对应的校验器
@Constraint(
        validatedBy = {NickValueConstraintValidator.class}
)
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE, ElementType.CONSTRUCTOR, ElementType.PARAMETER, ElementType.TYPE_USE})
@Retention(RetentionPolicy.RUNTIME)
public @interface NickValue {
    /**
     * 消息
     * 这里写该注解的全类名
     * 若校验失败，会前往ValidationMessages.properties来查询失败信息
     * 所以我们需要自己配置一个ValidationMessages.properties
     *
     * @return {@link String}
     */
    String message() default "{com.example.exceptiontest.valid.NickValue.message}";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    String[] vals() default {};
}
```
> 打印一下错误信息
```java
@ExceptionHandler(value = Throwable.class)
public Object handleException(Throwable e){
    // 打印错误信息
    log.error("错误:",e);
    HashMap<String, String> map = new HashMap<>();
    // 获取状态码
    map.put("code",String.valueOf(BizCodeEnume.UNKNOWN_EXCEPTION.getCode()));
    // 获取信息
    map.put("msg",BizCodeEnume.UNKNOWN_EXCEPTION.getMsg());
    return map;
}
```
> 指定分组
```java
@NickValue(vals={"喜羊羊","灰太狼"},groups = {AddGroup.class, DeleteGroup.class})
@TableField(value = "nick_name")
private String nick_name;
```
![image-20230218161238324](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181612517.png)
![image-20230218161251523](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181612589.png)
![image-20230218161306450](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181613521.png)
> 校验注解指定多个校验器
```java
// 这里可以添加多个校验器
@Constraint(
        validatedBy = {NickValueConstraintValidator.class}
)
```
> 此步骤参考[博客](https://blog.csdn.net/qq_37020594/article/details/122777649)
>
> 按住ctrl+H
![image-20230218162021761](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181620912.png)
可以看到存在很多校验器
> 需要注意的`问题`如下
**User**
![image-20230218162537252](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181625320.png)
**UserController**
![image-20230218164538925](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302181645029.png)
> 在修改用户的时候，我们是需要传入`密码`和`昵称`的
>
> 但是实际在个人主页修改个人昵称的时候，如果前端传递的user对象里只包含`用户id`和`昵称`而不包含密码，那么此时的校验无法通过
> 因为在`修改方法`内并未传入`密码`
> 但是也无法删除`密码`注解的UpdateGroup，否则修改密码的时候如果不传入`密码`也能通过校验
>
> 原因是修改昵称和修改密码调用的是`同一个修改方法`和`同一个UpdateGroup`
>
> 解决方法：`修改方法分开编写`，不要合并成同一个方法；同时创建一个新的修改分组，如`UpdateNiceNameGroup`
