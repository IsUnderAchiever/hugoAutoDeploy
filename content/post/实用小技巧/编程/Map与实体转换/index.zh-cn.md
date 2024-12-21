---
title: Map与实体转换
description: Map与实体转换
date: 2023-05-28
slug: Map与实体转换
image: 202412212036798.png
categories:
    - Java
---

# map与实体转换

> 以下工具类请查看[`此博客`](https://blog.csdn.net/weixin_50391597/article/details/129063508)
```java
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;
public class MapAndEntityConverter {
    /**
     * 利用反射将map集合封装成bean对象
     *
     * @param map
     * @param clazz
     * @return {@link T}
     * @throws Exception 异常
     */
    public static <T> T mapToBean(Map<String, Object> map, Class<?> clazz) throws Exception {
        Object obj = clazz.newInstance();
        if (map != null && !map.isEmpty() && map.size() > 0) {
            // map.entrySet():获取到Map集合中所有的键值对对象的集合(Set集合)
            for (Map.Entry<String, Object> entry : map.entrySet()) {
                // 属性名
                String propertyName = entry.getKey();    
                // 属性值
                Object value = entry.getValue();        
                String setMethodName = "set" + propertyName.substring(0, 1).toUpperCase() + propertyName.substring(1);
                //获取和map的key匹配的属性名称
                Field field = getClassField(clazz, propertyName);    
                if (field == null) {
                    continue;
                }
                Class<?> fieldTypeClass = field.getType();
                value = convertValType(value, fieldTypeClass);
                try {
                    clazz.getMethod(setMethodName, field.getType()).invoke(obj, value);
                } catch (NoSuchMethodException e) {
                    e.printStackTrace();
                }
            }
        }
        return (T) obj;
    }
    /**
     * 根据给定对象类匹配对象中的特定字段
     *
     * @param clazz     clazz
     * @param fieldName 字段名
     * @return {@link Field}
     */
    private static Field getClassField(Class<?> clazz, String fieldName) {
        if (Object.class.getName().equals(clazz.getName())) {
            return null;
        }
        Field[] declaredFields = clazz.getDeclaredFields();
        for (Field field : declaredFields) {
            if (field.getName().equals(fieldName)) {
                return field;
            }
        }
        //如果该类还有父类，将父类对象中的字段也取出
        Class<?> superClass = clazz.getSuperclass();    
        //递归获取
        if (superClass != null) {                        
            return getClassField(superClass, fieldName);
        }
        return null;
    }
    /**
     * 将map的value值转为实体类中字段类型匹配的方法
     *
     * @param value
     * @param fieldTypeClass
     * @return {@link Object}
     */
    private static Object convertValType(Object value, Class<?> fieldTypeClass) {
        Object retVal = null;
        if (Long.class.getName().equals(fieldTypeClass.getName())
                || long.class.getName().equals(fieldTypeClass.getName())) {
            retVal = Long.parseLong(value.toString());
        } else if (Integer.class.getName().equals(fieldTypeClass.getName())
                || int.class.getName().equals(fieldTypeClass.getName())) {
            retVal = Integer.parseInt(value.toString());
        } else if (Float.class.getName().equals(fieldTypeClass.getName())
                || float.class.getName().equals(fieldTypeClass.getName())) {
            retVal = Float.parseFloat(value.toString());
        } else if (Double.class.getName().equals(fieldTypeClass.getName())
                || double.class.getName().equals(fieldTypeClass.getName())) {
            retVal = Double.parseDouble(value.toString());
        } else {
            retVal = value;
        }
        return retVal;
    }
    /**
     * 对象转map
     *
     * @param obj obj
     * @return {@link Map}<{@link String}, {@link Object}>
     */
    public static Map<String, Object> objToMap(Object obj) {
        Map<String, Object> map = new HashMap<String, Object>();
        // 获取f对象对应类中的所有属性域
        Field[] fields = obj.getClass().getDeclaredFields();    
        for (int i = 0, len = fields.length; i < len; i++) {
            String varName = fields[i].getName();
            // 将key置为小写，默认为对象的属性
            varName = varName.toLowerCase();                    
            try {
                // 获取原来的访问控制权限
                boolean accessFlag = fields[i].isAccessible();    
                // 修改访问控制权限
                fields[i].setAccessible(true);                    
				// 获取在对象f中属性fields[i]对应的对象中的变量
                Object o = fields[i].get(obj);                    
                if (o != null) {
                    map.put(varName, o.toString());
                }
                // 恢复访问控制权限
                fields[i].setAccessible(accessFlag);            
            } catch (IllegalArgumentException | IllegalAccessException ex) {
                ex.printStackTrace();
            }
        }
        return map;
    }
}
```
**编写测试类**
```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UserToMap {
    private Long id;
    private String name;
    private String gender;
}
```
```java
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import java.util.HashMap;
import java.util.Map;
@Slf4j
@SpringBootTest
class DemoApplicationTests {
    @Test
    void contextLoads() {
    }
    /**
     * 实体转map测试
     */
    @Test
    void entityToMapTest() {
        UserToMap userToMap = new UserToMap(1L, "马小跳", "男");
        Map<String, Object> stringObjectMap = MapAndEntityConverter.objToMap(userToMap);
        System.out.println(stringObjectMap);
    }
    /**
     * map转实体测试
     */
    @Test
    void mapToEntityTest() {
        Map<String, Object> stringObjectMap = new HashMap<>();
        stringObjectMap.put("id", 1L);
        stringObjectMap.put("name", "马小跳");
        stringObjectMap.put("gender", "男");
        UserToMap user = null;
        try {
            user = MapAndEntityConverter.mapToBean(stringObjectMap, UserToMap.class);
        } catch (Exception e) {
            log.error("map转实体失败");
            throw new RuntimeException(e);
        }
        System.out.println(user);
    }
}
```
## 注意
> ### 有个需要注意lombok生成的class文件的get、set方法
>
> 有可能会导致转换出错`（博客楼主提供的工具类并未出错）`
>
> 视频[`查看`](https://www.bilibili.com/video/BV1vs4y1g763/?spm_id_from=333.337.search-card.all.click&vd_source=0de2cc95ad0adc3e9ec15f6dcede1a20)
>
> 视频中使用Jackson的ObjectMapper类实现map转entity
>
> entity有一个字段是lDate，而ObjectMapper的规定需要生成的get、set方法为`getlDate`、`setlDate`
>
> 但是lombok @Data注解生成的get、set方法为`getLDate`、`setLDate`
>
> 导致lDate转换后的值为空，同样的情况还出现在BeanUtils
>
> 解决方法就是手写（idea自动生成）get、set方法
楼主这里采用的set方法和lombok一致
```java
String setMethodName = "set" + propertyName.substring(0, 1).toUpperCase() + propertyName.substring(1);
```
有个需要注意的地方是entity转map，结果如下
```json
{uname=马小跳, gender=男, id=1}
```
而entity属性名为`uName`
