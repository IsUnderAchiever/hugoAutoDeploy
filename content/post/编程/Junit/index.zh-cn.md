---
title: Junit
description: Junit
date: 2024-02-16
slug: Junit
image: 202412212121680.png
categories:
    - Junit
---

# Junit测试
> 这里使用junit5进行演示
## 正常的情况
```java
/**
 * @Author: 不是菜鸡爱编程
 * @Date: 2023/11/29/21:30
 * @Description:
 */
public class Calculate {
    /**
     * 加
     *
     * @param i 我
     * @param j j
     * @return int
     */
    public static int add(int i,int j){
        return i+j;
    }
    /**
     * 减
     *
     * @param i 我
     * @param j j
     * @return int
     */
    public static int sub(int i,int j){
        return i-j;
    }
    /**
     * 乘
     *
     * @param i 我
     * @param j j
     * @return int
     */
    public static int multi(int i,int j){
        return i*j;
    }
    /**
     * 除
     *
     * @param i 我
     * @param j j
     * @return int
     */
    public static int div(int i,int j){
        return i/j;
    }
}
```
> 有以上类，我们需要进行测试
>
> 在idea中按住`Ctrl+Shift+T`
![image-20231129214419971](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202311292144145_repeat_1701265469164__211203.png)
> 勾选方法
![image-20231129214524901](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202311292145937_repeat_1701265524954__145173.png)
> 会生成以下模板
```java
class CalculateTest {
    @Test
    void add() {
    }
    @Test
    void sub() {
    }
    @Test
    void multi() {
    }
    @Test
    void div() {
    }
}
```
> 我们将使用断言`Assert`来完成测试
![image-20231129214629814](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202311292146882_repeat_1701265589895__016984.png)
> 我们将`预期值`与`实际值`进行比较
![image-20231129214713781](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202311292147830_repeat_1701265633841__794593.png)
![image-20231129214742612](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202311292147718_repeat_1701265662728__041266.png)
> 可以看到`100%测试覆盖率`
![image-20231129215021768](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202311292150876_repeat_1701265821886__706483.png)
> 注释掉一个测试方法后，导致测试覆盖率不足100%，因为有部分代码未被测试到
![image-20231129215132836](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202311292151937_repeat_1701265892946__483201.png)
> 修改预期值后导致测试方法不通过
## 异常的情况
> 在`Calculate`中添加一个方法`throwException`
```java
public class Calculate {
    // ...
	public static void throwException(){
        throw new IllegalStateException();
    }
}
```
> 可能抛出异常的方法该如何测试？
**测试方法1**
> 设置一个标志位
>
> 捕获到异常后，更改标志位
>
> 断言标志位为`True`
```java
    @Test
    void throwException() {
        boolean flag=false;
        try {
            Calculate.throwException();
        } catch (IllegalStateException e) {
            flag=true;
        }
        assertTrue(flag);
    }
```
**测试方法2**
> 后面是`lambda`表达式
```java
    @Test
    void throwException() {
        assertThrows(IllegalStateException.class, Calculate::throwException);
    }
```
---------
> junit4可使用如下写法
```java
@Test(expected=IndexOutOfBoundsException.class)
public void testIndexOutOfBoundsException() {
    ArrayList emptyList = new ArrayList();
    Object o = emptyList.get(0);
}
```
