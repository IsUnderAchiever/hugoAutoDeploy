---
title: 09_Java泛型
description: 09_Java泛型
date: 2023-04-01
slug: 09_Java泛型
image: 202412212036798.png
categories:
    - Java
---

Java泛型
-----------------------------------------------------
[参考视频](https://www.bilibili.com/video/BV1ds4y1E7uW/?spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=0de2cc95ad0adc3e9ec15f6dcede1a20)
### 在类上
> 这是个打印integer的类，但也仅仅只能打印integer了
>
> 难道以后要打印String、Double还需要复制粘贴再新建类吗？
```java
public class PrinterInteger {
    Integer needToPrint;
    public PrinterInteger(Integer needToPrint) {
        this.needToPrint = needToPrint;
    }
    public void print(){
        System.out.println(needToPrint);
    }
}
```
> 可以考虑采用以下方式
```java
public class Printer <T>{
    T needToPrint;
    public Printer(T needToPrint) {
        this.needToPrint = needToPrint;
    }
    public void print(){
        System.out.println(needToPrint);
    }
    /**
     * 测试
     */
    static class MyTest{
        public static void main(String[] args) {
            // 打印String
            Printer<String> stringPrinter = new Printer<>("hello world");
            stringPrinter.print();
            // 打印Double
            Printer<Double> doublePrinter = new Printer<>(30.5);
            doublePrinter.print();
        }
    }
}
```
> 泛型不能使用基本数据类型，如int、long等
> 泛型 extends
```java
public class Animal {
    String name;
    int age;
    public void eat(){
        System.out.println("吃饭ing");
    }
}
```
```java
public class Cat extends Animal{
    private String litterPreference;
    public Cat(String name) {
        this.name = name;
    }
    public Cat() {
    }
    public String getLitterPreference() {
        return litterPreference;
    }
    public void setLitterPreference(String litterPreference) {
        this.litterPreference = litterPreference;
    }
}
```
```java
public class Dog extends Animal{
    int walkDistancePreference;
}
```
> 泛型这么写
```java
public class Printer <T extends Animal>{
    T needToPrint;
    public Printer(T needToPrint) {
        this.needToPrint = needToPrint;
    }
    public void print(){
        System.out.println(needToPrint);
    }
}
```
![泛型继承](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152330491.png)
> 这个时候就会报错了，因为泛型写了`<T extends Animal>`，改写成如下写法后正确
```java
public class Printer <T extends Animal>{
    T needToPrint;
    public Printer(T needToPrint) {
        this.needToPrint = needToPrint;
    }
    public void print(){
        System.out.println(needToPrint);
    }
    /**
     * 测试
     */
    static class MyTest{
        public static void main(String[] args) {
            // 打印Cat
            Printer<Cat> stringPrinter = new Printer<>(new Cat());
            stringPrinter.print();
            // 打印Dog
            Printer<Dog> doublePrinter = new Printer<>(new Dog());
            doublePrinter.print();
        }
    }
}
```
> eat方法定义在Animal类中，但如果取消掉类上的`<T extends Animal>`后，这行代码会报错
![image-20230401130433407](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152330591.png)
![image-20230401130709488](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152331352.png)
> 但是java不支持多继承，所以如果还需要同时继承Animal怎么办，参考如下写法
>
> 但是注意，`接口要写在后面`，如果调换位置会报错
![image-20230401130833543](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152330482.png)
![image-20230401131024843](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152330279.png)
```java
public class Printer<T extends Animal & Serializable> {
    T needToPrint;
    public Printer(T needToPrint) {
        this.needToPrint = needToPrint;
    }
    public void print() {
        System.out.println(needToPrint);
    }
    /**
     * 测试
     */
    static class MyTest {
        public static void main(String[] args) {
        }
    }
}
```
> 多个泛型参数
```java
public class Printer<T, V> {
    T needToPrint;
    V moreMsg;
    public Printer(T needToPrint, V moreMsg) {
        this.needToPrint = needToPrint;
        this.moreMsg = moreMsg;
    }
    public void print() {
        System.out.printf("T:%s%n", needToPrint);
        System.out.printf("V:%s%n", moreMsg);
    }
    /**
     * 测试
     */
    static class MyTest {
        public static void main(String[] args) {
            Printer<String, Cat> catPrinter = new Printer<>("哈哈哈", new Cat());
            catPrinter.print();
        }
    }
}
```
### 在方法上
> 作用在方法上
![image-20230401131313812](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152330520.png)
> 注意，这个时候他并不知道T是一个泛型，你需要”告诉他”
![image-20230401131402488](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152331778.png)
```java
public class MyTest {
    public static void main(String[] args) {
        print("abc");
        print(123);
        print(new Cat());
    }
    public static <T> void print(T str){
        System.out.println(str+":打印完成");
    }
}
```
> 当然，也可以写多个参数
![image-20230401131824195](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152330133.png)
```java
public class MyTest {
    public static void main(String[] args) {
        print("哈哈哈", new Cat());
    }
    public static <T, V> void print(T str, V msg) {
        System.out.printf("T:%s%n", str);
        System.out.printf("V:%s%n", msg);
    }
}
```
> 指定返回值的泛型
```java
public class MyTest {
    public static void main(String[] args) {
        String print = print("哈哈哈", new Cat());
        System.out.println("返回值:"+print);
    }
    public static <T, V> T print(T str, V msg) {
        System.out.printf("T:%s%n", str);
        System.out.printf("V:%s%n", msg);
        return str;
    }
}
```
![image-20230401132537492](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152330061.png)
> 泛型通配符
**首先，来看一下下面这个问题**
![image-20230401133025790](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152331588.png)
**报错如下**
![image-20230401133013989](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152331449.png)
> **是不是觉得很奇怪？Integer是Object的子类，为什么不能这么传值呢?**
>
> 对，Integer是Object的子类没错，但是这里是`List<Object>`和`List<Integer>`
>
> `List<Integer>`并不是`List<Object>`的子类
>
> 或者可以把`List<Object>`换成Object，`List<Integer>`是`Object`的子类
>
> 但是我们要的不是这种效果
>
> 这时可以采用通配符`?`
```java
public class MyTest {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        print(list);
    }
    public static void print(List<?> list) {
        System.out.println(list);
    }
}
```
> 甚至还可以指定是否继承某个类
```java
public class MyTest {
    public static void main(String[] args) {        
        List<Cat> animal = new ArrayList<>();
        animal.add(new Cat());
        print(animal);
    }
    public static void print(List<? extends Animal> list) {
        System.out.println(list);
    }
}
```
![image-20230401133723105](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152331926.png)
