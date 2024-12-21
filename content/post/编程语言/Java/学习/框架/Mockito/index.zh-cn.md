---
title: Mockito
description: Mockito
date: 2024-03-01
slug: Mockito
image: 202412212122932.png
categories:
    - Mockito
---

# Mockito
## 基础
> 当我们`mock`了某个对象后，调用该方法的时候是并不会去执行实际方法的逻辑的
```java
public class User {
    private String name = "小张";
    private String gender = "男";
    private Integer age = 15;
}
//=================================
class MockitoTest {
    /**
     * 函数是否已执行
     */
    @Test
    void functionHasExecutedTest() {
        // 模拟user对象
        User mockUser = mock(User.class);
        mockUser.setName("呵呵呵");
        mockUser.setGender("女");
        mockUser.setAge(11);
        System.out.println("mockUser.getName() = " + mockUser.getName());
        System.out.println("mockUser.getGender() = " + mockUser.getGender());
        System.out.println("mockUser.getAge() = " + mockUser.getAge());
    }
}
```
![image-20240303175851275](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202403032042560.png)
> 查看覆盖率发现对应的`get`、`set`方法没有被调用过
>
> 这也就是`调用mock对象方法时，不会执行实际逻辑`的意思
### 验证方法被调用的次数
```java
import org.junit.jupiter.api.Test;
import java.util.List;
import static org.mockito.Mockito.*;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    /**
     * 函数是否已执行
     */
    @Test
    void functionHasExecutedTest(){
        // 模拟创建List对象
        List mockList = mock(List.class);
        // add("星期一")方法
        mockList.add("星期一");
        // 校验add("星期一")方法的执行次数是否是1
        // times(1)表示该方法执行了一次
        verify(mockList,times(1)).add("星期一");
    }
}
```
> 在`Mocktio 4.10.0`中还可以使用`List mockList = mock();`来mock对象，无需传入对应`class`
### 模拟方法返回值
```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.junit.jupiter.api.Test;
import com.example.entity.User;
import static org.mockito.Mockito.mock;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    /**
     * 函数是否已执行
     */
    @Test
    void functionHasExecutedTest() {
        // 真实user对象
        User user = new User();
        // 打印为小张
        System.out.println("user.getName() = " + user.getName());
        // 模拟user对象
        User mockUser = mock(User.class);
        // 打印为null，因为还没mock方法的返回值
        System.out.println("mockUser.getName() = " + mockUser.getName());
    }
}
```
> mock方法的返回值
>
> `when(object.functionName()).thenReturn(result)`来mock方法的返回值
```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.junit.jupiter.api.Test;
import com.example.entity.User;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    /**
     * 函数是否已执行
     */
    @Test
    void functionHasExecutedTest() {
        // 真实user对象
        User user = new User();
        // 打印为小张
        System.out.println("user.getName() = " + user.getName());
        // 模拟user对象
        User mockUser = mock(User.class);
        // mock方法的返回值
        when(mockUser.getName()).thenReturn("小明");
        // 打印为小明
        System.out.println("mockUser.getName() = " + mockUser.getName());
    }
}
```
> 当调用`mockUser.getName()`方法时，会返回`小明`
> 因为我们mock了该方法的返回值，所以并不会去走实际逻辑，而是直接返回我们`mock`的返回值`小明`
### 模拟异常
> mock异常
>
> `when(object.functionName()).thenThrow(throwable);`来mock异常
```java
import com.example.entity.User;
import org.junit.jupiter.api.Test;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    /**
     * 函数是否已执行
     */
    @Test
    void functionHasExecutedTest() {
        // 模拟user对象
        User mockUser = mock(User.class);
        when(mockUser.getAge()).thenThrow(new RuntimeException("年龄保密"));
        System.out.println("mockUser.getAge() = " + mockUser.getAge());
    }
}
```
> 使用`doThrow`来mock方法体的异常
>
> `doThrow(throwable).when(mockObject).functionName();`
```java
import com.example.entity.User;
import org.junit.jupiter.api.Test;
import static org.mockito.Mockito.doThrow;
import static org.mockito.Mockito.mock;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    /**
     * 函数是否已执行
     */
    @Test
    void functionHasExecutedTest() {
        // 模拟user对象
        User mockUser = mock(User.class);
        doThrow(new RuntimeException("年龄保密")).when(mockUser).getAge();
        System.out.println("mockUser.getAge() = " + mockUser.getAge());
    }
}
```
> `Mocktio`可以配合`Junit`来使用**更加优雅**
```java
import com.example.entity.User;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.mockito.Mockito.doThrow;
import static org.mockito.Mockito.mock;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    /**
     * 函数是否已执行
     */
    @Test
    void functionHasExecutedTest() {
        // 模拟user对象
        User mockUser = mock(User.class);
        doThrow(new RuntimeException("年龄保密")).when(mockUser).getAge();
        // 断言异常类型
        RuntimeException runtimeException = assertThrows(RuntimeException.class, () -> {
            // 发生异常的代码块
            System.out.println("mockUser.getAge() = " + mockUser.getAge());
        });
        // 断言异常信息
        assertEquals("年龄保密",runtimeException.getMessage());
    }
}
```
### 更多细节
```java
class MockitoTest {
    int generateInt(int num){
        return num;
    }
}
```
> 希望`generateInt`方法传入任意值，都返回`1`
>
> `anyInt()`
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.ArgumentMatchers.anyInt;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    /**
     * 函数是否已执行
     */
    @Test
    void functionHasExecutedTest() {
        MockitoTest mockitoTest=mock(MockitoTest.class);
        when(mockitoTest.generateInt(anyInt())).thenReturn(1);
        assertEquals(mockitoTest.generateInt(2),1);
    }
    int generateInt(int num){
        return num;
    }
}
```
> `atLeast(xxx)`、`atMost(xxx)`mock方法调用的最小、最大次数
>
> `verify(mockitoObject,atLeast(num)).functionName();`
```java
import org.junit.jupiter.api.Test;
import static org.mockito.ArgumentMatchers.anyInt;
import static org.mockito.Mockito.*;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    /**
     * 函数是否已执行
     */
    @Test
    void functionHasExecutedTest() {
        MockitoTest mockitoTest=mock(MockitoTest.class);
        // 调用三次该方法
        mockitoTest.generateInt(1);
        mockitoTest.generateInt(2);
        mockitoTest.generateInt(3);
        verify(mockitoTest,atLeast(3)).generateInt(anyInt());
    }
    int generateInt(int num){
        return num;
    }
}
```
> `when(mockitoTest.generateInt(anyInt())).thenReturn(2).thenReturn(1)`支持链式调用
>
> 表示第一次调用`generateInt`返回2，第二次调用返回1
>
> 配合`Junit`断言使用，断言不一致则抛出`AssertionFailedError`异常
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.ArgumentMatchers.anyInt;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    /**
     * 函数是否已执行
     */
    @Test
    void functionHasExecutedTest() {
        MockitoTest mockitoTest = mock(MockitoTest.class);
        when(mockitoTest.generateInt(anyInt()))
                // 第一次调用返回2
                .thenReturn(2)
                // 第二次调用返回1
                .thenReturn(1);
        assertEquals(2,mockitoTest.generateInt(1));
        assertEquals(1,mockitoTest.generateInt(2));
    }
    int generateInt(int num) {
        return num;
    }
}
```
> 验证执行顺序
>
> 将需要排序的对象传入`inOrder()`方法中
>
> 案例中`add()方法`和`verify()`的顺序不能反，一定要先执行模拟方法`add`，然后再`verify `校验顺序
```java
import org.junit.jupiter.api.Test;
import org.mockito.InOrder;
import java.util.List;
import static org.mockito.Mockito.inOrder;
import static org.mockito.Mockito.mock;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    /**
     * 函数是否已执行
     */
    @Test
    void functionHasExecutedTest() {
        List mockitoList1 = mock(List.class);
        List mockitoList2 = mock(List.class);
        mockitoList1.add(1);
        mockitoList2.add(1);
        mockitoList1.add(2);
        InOrder inOrder = inOrder(mockitoList1, mockitoList2);
        inOrder.verify(mockitoList1).add(1);
        inOrder.verify(mockitoList2).add(1);
        inOrder.verify(mockitoList1).add(2);
    }
}
```
> 顺序不一致，则会抛出以下异常信息
```error
org.mockito.exceptions.verification.VerificationInOrderFailure: 
Verification in order failure
Wanted but not invoked:
list.add(1);
-> at com.example.MockitoTest.functionHasExecutedTest(MockitoTest.java:30)
Wanted anywhere AFTER following interaction:
list.add(2);
-> at com.example.MockitoTest.functionHasExecutedTest(MockitoTest.java:26)
```
## 注解
### @Mock
> `@Mock`代替`Mockito.mock()`方法
```java
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.springframework.boot.test.context.SpringBootTest;
import java.util.List;
import static org.mockito.Mockito.when;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
@SpringBootTest
class MockitoTest {
    @Mock
    List<Integer> mockitoList;
    /**
     * 函数是否已执行
     */
    @Test
    void mockitoTest() {
        when(mockitoList.get(0)).thenReturn(1);
        System.out.println("mockitoList.get(0) = " + mockitoList.get(0));
    }
}
```
> 注意`@SpringBootTest`
>
> 或者使用如下写法
```java
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import java.util.List;
import static org.mockito.Mockito.when;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    private AutoCloseable closeable;
    @Mock
    List<Integer> mockitoList;
    @BeforeEach
    void init() {
        closeable = MockitoAnnotations.openMocks(this);
    }
    @AfterEach
    void destroy() {
        try {
            closeable.close();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 函数是否已执行
     */
    @Test
    void mockitoTest() {
        when(mockitoList.get(0)).thenReturn(1);
        System.out.println("mockitoList.get(0) = " + mockitoList.get(0));
    }
}
```
### @Spy
> @Spy用于返回一个真实的对象，@Mock返回的是一个模拟的假对象
>
> 除非使用`when(xxx).thenReturn(xxx)`为其设置返回值，否则将会执行真实的业务逻辑
```java
import com.example.entity.User;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.MockitoAnnotations;
import org.mockito.Spy;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    private AutoCloseable closeable;
    @Spy
    User mockUser;
    @BeforeEach
    void init() {
        closeable = MockitoAnnotations.openMocks(this);
    }
    @AfterEach
    void destroy() {
        try {
            closeable.close();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 函数是否已执行
     */
    @Test
    void mockitoTest() {
        System.out.println("mockUser.getName() = " + mockUser.getName());
    }
}
```
![image-20240303201902361](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202403032043970.png)
> 可以看到这里执行了`getName`方法的真实逻辑
### @Captor
> 捕获方法的参数，进行进一步的校验
```java
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.ArgumentCaptor;
import org.mockito.Captor;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import java.util.Map;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    private AutoCloseable closeable;
    @Mock
    Map<Integer,String> mockMap;
    @Captor
    ArgumentCaptor<Integer> key;
    @Captor
    ArgumentCaptor<String> value;
    @BeforeEach
    void init() {
        closeable = MockitoAnnotations.openMocks(this);
    }
    @AfterEach
    void destroy() {
        try {
            closeable.close();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 函数是否已执行
     */
    @Test
    void mockitoTest() {
        mockMap.put(1,"星期一");
        verify(mockMap,times(1)).put(key.capture(),value.capture());
        assertEquals(1,key.getValue());
        assertEquals("星期一",value.getValue());
    }
}
```
### @InjectMocks
> `@InjectMocks`注解可以注入指定的`Bean`，但是和`@Autowired`有些区别
```java
@Component
public class UserMapper {
    public User selectUserByName(String name){
        User user = new User();
        user.setName(name);
        System.out.println("mapper真实逻辑已执行");
        return user;
    }
}
```
```java
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
    public User getUserByName(String name){
        System.out.println("service真实逻辑已执行");
        return userMapper.selectUserByName(name);
    }
}
```
> 这样注入`UserService`在执行时会报错`NullPointerException`
```java
import com.example.entity.User;
import com.example.service.UserService;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.MockitoAnnotations;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    private AutoCloseable closeable;
    @InjectMocks
    UserService userService;
    @BeforeEach
    void init() {
        closeable = MockitoAnnotations.openMocks(this);
    }
    @AfterEach
    void destroy() {
        try {
            closeable.close();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 函数是否已执行
     */
    @Test
    void mockitoTest() {
        User userByName = userService.getUserByName("李四");
        System.out.println("userByName = " + userByName);
    }
}
```
> 因为此时只注入了`UserService`，而`UserMapper`并没有被注入
>
> 请看如下两种写法
>
> 使用`@Spy`来mock`UserMapper`
>
> 使用`@Mock`来mock`UserMapper`
```java
import com.example.entity.User;
import com.example.mapper.UserMapper;
import com.example.service.UserService;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.MockitoAnnotations;
import org.mockito.Spy;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    private AutoCloseable closeable;
    @InjectMocks
    UserService userService;
    @Spy
    UserMapper userMapper;
    @BeforeEach
    void init() {
        closeable = MockitoAnnotations.openMocks(this);
    }
    @AfterEach
    void destroy() {
        try {
            closeable.close();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 函数是否已执行
     */
    @Test
    void mockitoTest() {
        User userByName = userService.getUserByName("李四");
        System.out.println("userByName = " + userByName);
    }
}
```
> 以上代码会执行`UserMapper`的`getUserByName`方法的真实逻辑
```java
import com.example.entity.User;
import com.example.mapper.UserMapper;
import com.example.service.UserService;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import static org.mockito.Mockito.when;
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2024/03/03/17:22
 * @Description:
 */
class MockitoTest {
    private AutoCloseable closeable;
    @InjectMocks
    UserService userService;
    @Mock
    UserMapper userMapper;
    @BeforeEach
    void init() {
        closeable = MockitoAnnotations.openMocks(this);
    }
    @AfterEach
    void destroy() {
        try {
            closeable.close();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 函数是否已执行
     */
    @Test
    void mockitoTest() {
        User mockUser = new User();
        mockUser.setName("张三");
        // mock方法返回值
        when(userMapper.selectUserByName("李四")).thenReturn(mockUser);
        User userByName = userService.getUserByName("李四");
        System.out.println("userByName = " + userByName);
    }
}
```
> 而以上代码并不会去执行真实逻辑
>
> 而是返回mock的返回值`张三`
