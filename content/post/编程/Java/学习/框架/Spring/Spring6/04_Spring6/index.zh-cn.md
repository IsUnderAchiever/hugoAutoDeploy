---
title: 04_Spring6
description: 04_Spring6
date: 2023-03-14
slug: 04_Spring6
image: 202412212133331.png
categories:
    - Spring
---

Spring6学习04
====================================================================
手写IOC
--------------------------------------------------
> 实现过程
> 
> 1.  创建子模块
> 2.  创建测试类
> 3.  创建两个注解
>     1.  @Bean(代替@Component)
>     2.  @Di(代替@Autowired)
> 4.  创建Bean容器接口ApplicationContext，定义方法，返回对象
> 5.  实现Bean容器接口
>     1.  返回对象
>     2.  根据包规则加载bean
> 
> **`规则:扫描当前包及其子包里的所有类，判断类上是否有@Bean注解，如果有，则把这个类通过反射实例化`**
### 第一步，搭建项目
**创建两个注解，@Bean、@Di**
```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Bean {
}
```
```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Di {
}
```
**使用如下方式进行注入**
```java
@Bean
public class UserServiceImpl implements UserService {
    @Di
    private UserDao userDao;
}
```
**新建ApplicationContext接口**
```java
public interface ApplicationContext {
    Object getBean(Class<?> clazz);
}
```
**对接口进行实例化**
```java
import java.util.HashMap;
import java.util.Map;
public class AnnotationApplicationContext implements ApplicationContext{
    // 创建Map集合，用于存放Bean对象
    private Map<Class<?>,Object> beanFactory=new HashMap<>();
    /**
     * 返回bean对象
     */
    @Override
    public Object getBean(Class<?> clazz) {
        return beanFactory.get(clazz);
    }
    /**
     * 创建有参构造方法，传递包路径，设置包扫描规则
     * 当前包及其子包，标注有@Bean注解，把这个类通过反射实例化
     *
     * @param basePackage 包路径
     */
    public AnnotationApplicationContext(String basePackage) {
        //
    }
}
```
> 为什么要写`AnnotationApplicationContext`的构造方法？
> 
> 我们先来看看原本Spring是怎么写的
![image-20230409104422741](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152335347.png)
**以下是暂时的项目结构**
![image-20230409104055973](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152335120.png)
### 第二步，配置扫描规则
> 用法应该如下所示
```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationApplicationContext("com.example");
        Object bean = context.getBean(UserDaoImpl.class);
    }
}
```
1.  将.替换成\\ `（com.example）->（com\example）`
2.  获取包的(带盘符的)对路径 `（这里的绝对路径并非是当前类的绝对路径，而是编译后生成的target内的类路径）`
> 由于转义字符的影响，所以得这么写
> 
> 如果不放心，可以试着打印输出一下
```java
String packagePath = basePackage.replaceAll("\\.", "\\\\");
```
![image-20230409112600074](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152335485.png)
```java
public class AnnotationApplicationContext implements ApplicationContext {
    // 创建Map集合，用于存放Bean对象
    private Map<Class<?>, Object> beanFactory = new HashMap<>();
    private String rootPath;
    /**
     * 返回bean对象
     */
    @Override
    public Object getBean(Class<?> clazz) {
        return beanFactory.get(clazz);
    }
    /**
     * 创建有参构造方法，传递包路径，设置包扫描规则
     * 当前包及其子包，标注有@Bean注解，把这个类通过反射实例化
     *
     * @param basePackage 包路径
     */
    public AnnotationApplicationContext(String basePackage) {
        // com.example
        // 1. 将.替换成\
        String packagePath = basePackage.replaceAll("\\.", "\\\\");
        try {
            // 2. 获取包的绝对路径
            Enumeration<URL> urls = Thread.currentThread().getContextClassLoader().getResources(packagePath);
            // 遍历枚举对象
            while (urls.hasMoreElements()) {
                URL url = urls.nextElement();
                // 斜杠会做url编码，所以要进行转码
                String filePath = URLDecoder.decode(url.getFile(), "UTF-8");
                // 获取包前面的路径部分，字符串截取
                rootPath=filePath.substring(0,filePath.length()-packagePath.length());
                // 包扫描
                loadBean(new File(filePath));
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 包扫描过程
     * 实例化
     */
    private void loadBean(File file) {
        // 1.判断当前是否是文件夹
        // 2.获取文件夹里的所有内容(文件夹、文件)
        // 3.若文件夹里为空，直接返回
        // 4.文件夹不为空，遍历文件夹所有内容
        // 4.1遍历得到每个File对象，递归判断文件夹里是否还有文件夹...
        // 4.2遍历得到File对象不是文件夹，是文件
        // 4.3得到【包路径+类名称】部分-字符串截取
        // 4.4判断当前类型是否是.class
        // 4.5如果是.class类型，就把\替换成. 并 去掉后缀.class
        // com.example.service.UserServiceImpl  因为只有获取类全路径名才能使用反射
        // 4.6判断类上是否有注解@Bean，如果有，实例化过程
        // 4.7把对象实例化之后，放到map集合beanFactory
        // 小细节:如果有接口，就放接口class作为key，如果没有，就让自己class作为key
    }
}
```
### 第三步，配置包扫描过程
> ```
> 步骤
> ```
>
> 1.  判断当前是否是文件夹
> 2.  获取文件夹里的所有内容(文件夹、文件)
> 3.  若文件夹里为空，直接返回
> 4.  文件夹不为空，遍历文件夹所有内容  
>     4.1 遍历得到每个File对象，递归判断文件夹里是否还有文件夹…  
>     4.2 遍历得到File对象不是文件夹，是文件  
>     4.3 得到【包路径+类名称】部分-字符串截取  
>     4.4 判断当前类型是否是.class  
>     4.5 如果是.class类型，就把\\替换成. 并 去掉后缀.class `com.example.service.UserServiceImpl 因为只有获取类全路径名才能使用反射`  
>     4.6 判断类上是否有注解@Bean，如果有，实例化过程  
>     4.7 把对象实例化之后，放到map集合beanFactory  
>     小细节:如果有接口，就放接口class作为key，如果没有，就让自己class作为key
```java
public class AnnotationApplicationContext implements ApplicationContext {
    // 创建Map集合，用于存放Bean对象
    private Map<Class<?>, Object> beanFactory = new HashMap<>();
    private String rootPath;
    /**
     * 返回bean对象
     */
    @Override
    public Object getBean(Class<?> clazz) {
        return beanFactory.get(clazz);
    }
    /**
     * 创建有参构造方法，传递包路径，设置包扫描规则
     * 当前包及其子包，标注有@Bean注解，把这个类通过反射实例化
     *
     * @param basePackage 包路径
     */
    public AnnotationApplicationContext(String basePackage) {
        // com.example
        // 1. 将.替换成\
        String packagePath = basePackage.replaceAll("\\.", "\\\\");
        try {
            // 2. 获取包的绝对路径
            Enumeration<URL> urls = Thread.currentThread().getContextClassLoader().getResources(packagePath);
            // 遍历枚举对象
            while (urls.hasMoreElements()) {
                URL url = urls.nextElement();
                // 斜杠会做url编码，所以要进行转码
                String filePath = URLDecoder.decode(url.getFile(), "UTF-8");
                // 获取包前面的路径部分，字符串截取
                rootPath=filePath.substring(0,filePath.length()-packagePath.length());
                // 包扫描
                loadBean(new File(filePath));
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 包扫描过程
     * 实例化
     */
    private void loadBean(File file) throws Exception {
        // 1.判断当前是否是文件夹
        if(file.isDirectory()){
            // 2.获取文件夹里的所有内容(文件夹、文件)
            File[] childrenFiles = file.listFiles();
            // 3.若文件夹里为空，直接返回
            if(childrenFiles==null||childrenFiles.length==0){
                return;
            }
            // 4.文件夹不为空，遍历文件夹所有内容
            for(File child:childrenFiles){
                // 4.1遍历得到每个File对象，递归判断文件夹里是否还有文件夹...
                if(child.isDirectory()){
                    // 递归
                    loadBean(child);
                }else{
                    // 4.2遍历得到File对象不是文件夹，是文件
                    // 4.3得到【包路径+类名称】部分-字符串截取
                    String pathWithClass = child.getAbsolutePath().substring(rootPath.length() - 1);
                    // 4.4判断当前类型是否是.class
                    if(pathWithClass.contains(".class")){
                        // 4.5如果是.class类型，就把\替换成. 并 去掉后缀.class
                        // com.example.service.UserServiceImpl  因为只有获取类全路径名才能使用反射
                        String allName = pathWithClass.replaceAll("\\\\", ".").replace(".class", "");
                        // 4.6判断类上是否有注解@Bean，如果有，实例化过程
                        // 4.6.1获取类的class对象
                        Class<?> clazz = Class.forName(allName);
                        // 4.6.2判断，如果不是接口，则实例化
                        if(!clazz.isInterface()){
                            // 4.6.3判断，类上是否有注解@Bean
                            Bean annotation = clazz.getAnnotation(Bean.class);
                            if(annotation!=null){
                                // 类上@Bean有注解
                                // 4.6.4实例化
                                Object instance = clazz.getConstructor().newInstance();
                                // 4.7把对象实例化之后，放到map集合beanFactory
                                // 4.7.1小细节:如果有接口，就放接口class作为key，如果没有，就让自己class作为key
                                if(clazz.getInterfaces().length>0){
                                    beanFactory.put(clazz.getInterfaces()[0],instance);
                                }else{
                                    beanFactory.put(clazz,instance);
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```
### 第四步，测试@Bean注解
```java
public interface UserDao {
    void insert();
}
```
```java
@Bean
public class UserDaoImpl implements UserDao {
    @Override
    public void insert() {
        System.out.println("userDao--insert");
    }
}
```
```java
public interface UserService {
    void add();
}
```
```java
@Bean
public class UserServiceImpl implements UserService {
    @Di
    private UserDao userDao;
    @Override
    public void add() {
        System.out.println("service--add");
        // TODO 调用dao的方法
    }
}
```
```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationApplicationContext("com.example");
        UserService userService = (UserService) context.getBean(UserService.class);
        System.out.println(userService);
        userService.add();
    }
}
```
### 第五步，编写@Di注解属性注入逻辑
> 实例化对象在beanFactory的map集合里  
> 1.遍历beanFactory的map集合  
> 2.获取map集合每个对象(value)，每个对象的属性获取到  
> 3.遍历得到每个对象属性数组，得到每个属性  
> 4.属性上是否有@Di的注解(如果是私有属性，需要设置为【可以设置值】)  
> 5.如果有@Di的注解，把对象注入
```java
public class AnnotationApplicationContext implements ApplicationContext {
    // 创建Map集合，用于存放Bean对象
    private Map<Class<?>, Object> beanFactory = new HashMap<>();
    private String rootPath;
    /**
     * 返回bean对象
     */
    @Override
    public Object getBean(Class<?> clazz) {
        return beanFactory.get(clazz);
    }
    /**
     * 创建有参构造方法，传递包路径，设置包扫描规则
     * 当前包及其子包，标注有@Bean注解，把这个类通过反射实例化
     *
     * @param basePackage 包路径
     */
    public AnnotationApplicationContext(String basePackage) {
        // com.example
        // 1. 将.替换成\
        String packagePath = basePackage.replaceAll("\\.", "\\\\");
        try {
            // 2. 获取包的绝对路径
            Enumeration<URL> urls = Thread.currentThread().getContextClassLoader().getResources(packagePath);
            // 遍历枚举对象
            while (urls.hasMoreElements()) {
                URL url = urls.nextElement();
                // 斜杠会做url编码，所以要进行转码
                String filePath = URLDecoder.decode(url.getFile(), "UTF-8");
                // 获取包前面的路径部分，字符串截取
                rootPath = filePath.substring(0, filePath.length() - packagePath.length());
                // 包扫描
                loadBean(new File(filePath));
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        // 属性注入
        loadDi();
    }
    /**
     * 属性注入
     */
    private void loadDi() {
        // 实例化对象在beanFactory的map集合里
        // 1.遍历beanFactory的map集合
        Set<Map.Entry<Class<?>, Object>> entries = beanFactory.entrySet();
        for (Map.Entry<Class<?>, Object> entry : entries) {
            // 2.获取map集合每个对象(value)，每个对象的属性获取到
            Object obj = entry.getValue();
            // 获取对象class
            Class<?> clazz = obj.getClass();
            // 获取每个对象中的属性
            Field[] declaredFields = clazz.getDeclaredFields();
            // 3.遍历得到每个对象属性数组，得到每个属性
            for (Field field : declaredFields) {
                // 4.属性上是否有@Di的注解(如果是私有属性，需要设置为【可以设置值】)
                Di annotation = field.getAnnotation(Di.class);
                if (annotation != null) {
                    // 属性上有@Di注解
                    // 如果是私有属性，需要设置为【可设置值】
                    field.setAccessible(true);
                    // 5.如果有@Di的注解，把对象注入
                    System.out.println("类型:"+field.getType());
                    try {
                        field.set(obj, beanFactory.get(field.getType()));
                    } catch (IllegalAccessException e) {
                        throw new RuntimeException(e);
                    }
                }
            }
        }
    }
    /**
     * 包扫描过程
     * 实例化
     */
    private void loadBean(File file) throws Exception {
        // 1.判断当前是否是文件夹
        if (file.isDirectory()) {
            // 2.获取文件夹里的所有内容(文件夹、文件)
            File[] childrenFiles = file.listFiles();
            // 3.若文件夹里为空，直接返回
            if (childrenFiles == null || childrenFiles.length == 0) {
                return;
            }
            // 4.文件夹不为空，遍历文件夹所有内容
            for (File child : childrenFiles) {
                // 4.1遍历得到每个File对象，递归判断文件夹里是否还有文件夹...
                if (child.isDirectory()) {
                    // 递归
                    loadBean(child);
                } else {
                    // 4.2遍历得到File对象不是文件夹，是文件
                    // 4.3得到【包路径+类名称】部分-字符串截取
                    String pathWithClass = child.getAbsolutePath().substring(rootPath.length() - 1);
                    // 4.4判断当前类型是否是.class
                    if (pathWithClass.contains(".class")) {
                        // 4.5如果是.class类型，就把\替换成. 并 去掉后缀.class
                        // com.example.service.UserServiceImpl  因为只有获取类全路径名才能使用反射
                        String allName = pathWithClass.replaceAll("\\\\", ".").replace(".class", "");
                        // 4.6判断类上是否有注解@Bean，如果有，实例化过程
                        // 4.6.1获取类的class对象
                        Class<?> clazz = Class.forName(allName);
                        // 4.6.2判断，如果不是接口，则实例化
                        if (!clazz.isInterface()) {
                            // 4.6.3判断，类上是否有注解@Bean
                            Bean annotation = clazz.getAnnotation(Bean.class);
                            if (annotation != null) {
                                // 类上@Bean有注解
                                // 4.6.4实例化
                                Object instance = clazz.getConstructor().newInstance();
                                // 4.7把对象实例化之后，放到map集合beanFactory
                                // 4.7.1小细节:如果有接口，就放接口class作为key，如果没有，就让自己class作为key
                                if (clazz.getInterfaces().length > 0) {
                                    beanFactory.put(clazz.getInterfaces()[0], instance);
                                } else {
                                    beanFactory.put(clazz, instance);
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```
```java
@Bean
public class UserServiceImpl implements UserService {
    @Di
    private UserDao userDao;
    @Override
    public void add() {
        System.out.println("service--add");
        // 调用dao的方法
        userDao.insert();
    }
}
```
![image-20230409150523624](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152336525.png)
* * *
