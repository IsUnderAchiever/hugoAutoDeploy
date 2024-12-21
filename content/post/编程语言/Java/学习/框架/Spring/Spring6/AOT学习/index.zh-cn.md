---
title: AOT学习
description: AOT学习
date: 2023-03-16
slug: AOT学习
image: 202412212133331.png
categories:
    - Spring
---

AOT学习
=====
AOT学习
--------------------------------------------------
> [选自 尚硅谷Spring6教程](https://www.bilibili.com/video/BV1kR4y1b7Qc/?spm_id_from=333.337.search-card.all.click&vd_source=0de2cc95ad0adc3e9ec15f6dcede1a20)
### JIT
> just in time，动态编译(实时编译)，边运行边编译
> 
> 在程序运行时候，动态生成代码
> 
> 启动时比较慢，编译时需要占用运行时候资源
> 
> 总结：程序运行过程中，把字节码转换成硬盘上直接运行的机器码，部署到环境的过程
### AOT
> ahead of time，运行前编译，提前编译
> 
> 可以把源代码直接转换成机器码，启动快，内存占用低
> 
> `缺点:`运行时不能优化，程序安装时间过长
> 
> 总结：在程序运行之前，就把字节码转换成机器码
#### 11.1.1、JIT与AOT的区别
JIT和AOT 这个名词是指两种不同的编译方式，这两种编译方式的主要区别在于是否在“运行时”进行编译
**（1）JIT， Just-in-time,动态(即时)编译，边运行边编译；**
在程序运行时，根据算法计算出热点代码，然后进行 JIT 实时编译，这种方式吞吐量高，有运行时性能加成，可以跑得更快，并可以做到动态生成代码等，但是相对启动速度较慢，并需要一定时间和调用频率才能触发 JIT 的分层机制。JIT 缺点就是编译需要占用运行时资源，会导致进程卡顿。
**（2）AOT，Ahead Of Time，指运行前编译，预先编译。**
AOT 编译能直接将源代码转化为机器码，内存占用低，启动速度快，可以无需 runtime 运行，直接将 runtime 静态链接至最终的程序中，但是无运行时性能加成，不能根据程序运行情况做进一步的优化，AOT 缺点就是在程序运行前编译会使程序安装的时间增加。
**简单来讲：**JIT即时编译指的是在程序的运行过程中，将字节码转换为可在硬件上直接运行的机器码，并部署至托管环境中的过程。而 AOT 编译指的则是，在程序运行之前，便将字节码转换为机器码的过程。
```
.java -> .class -> (使用jaotc编译工具) -> .so（程序函数库,即编译好的可以供其他程序使用的代码和数据）
```
![image-20221207113544080](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048128.png)
**（3）AOT的优点**
**简单来讲，**Java 虚拟机加载已经预编译成二进制库，可以直接执行。不必等待及时编译器的预热，减少 Java 应用给人带来“第一次运行慢” 的不良体验。
在程序运行前编译，可以避免在运行时的编译性能消耗和内存消耗  
可以在程序运行初期就达到最高性能，程序启动速度快  
运行产物只有机器码，打包体积小
**AOT的缺点**
由于是静态提前编译，不能根据硬件情况或程序运行情况择优选择机器指令序列，理论峰值性能不如JIT  
没有动态能力，同一份产物不能跨平台运行
第一种即时编译 (JIT) 是默认模式，Java Hotspot 虚拟机使用它在运行时将字节码转换为机器码。后者提前编译 (AOT)由新颖的 GraalVM 编译器支持，并允许在构建时将字节码直接静态编译为机器码。
现在正处于云原生，降本增效的时代，Java 相比于 Go、Rust 等其他编程语言非常大的弊端就是启动编译和启动进程非常慢，这对于根据实时计算资源，弹性扩缩容的云原生技术相冲突，Spring6 借助 AOT 技术在运行时内存占用低，启动速度快，逐渐的来满足 Java 在云原生时代的需求，对于大规模使用 Java 应用的商业公司可以考虑尽早调研使用 JDK17，通过云原生技术为公司实现降本增效。
#### 11.1.2、Graalvm
Spring6 支持的 AOT 技术，这个 GraalVM 就是底层的支持，Spring 也对 GraalVM 本机映像提供了一流的支持。GraalVM 是一种高性能 JDK，旨在加速用 Java 和其他 JVM 语言编写的应用程序的执行，同时还为 JavaScript、Python 和许多其他流行语言提供运行时。 GraalVM 提供两种运行 Java 应用程序的方法：在 HotSpot JVM 上使用 Graal 即时 (JIT) 编译器或作为提前 (AOT) 编译的本机可执行文件。 GraalVM 的多语言能力使得在单个应用程序中混合多种编程语言成为可能，同时消除了外语调用成本。GraalVM 向 HotSpot Java 虚拟机添加了一个用 Java 编写的高级即时 (JIT) 优化编译器。
GraalVM 具有以下特性：
（1）一种高级优化编译器，它生成更快、更精简的代码，需要更少的计算资源
（2）AOT 本机图像编译提前将 Java 应用程序编译为本机二进制文件，立即启动，无需预热即可实现最高性能
（3）Polyglot 编程在单个应用程序中利用流行语言的最佳功能和库，无需额外开销
（4）高级工具在 Java 和多种语言中调试、监视、分析和优化资源消耗
总的来说对云原生的要求不算高短期内可以继续使用 2.7.X 的版本和 JDK8，不过 Spring 官方已经对 Spring6 进行了正式版发布。
#### 11.1.3、Native Image
目前业界除了这种在JVM中进行AOT的方案，还有另外一种实现Java AOT的思路，那就是直接摒弃JVM，和C/C++一样通过编译器直接将代码编译成机器代码，然后运行。这无疑是一种直接颠覆Java语言设计的思路，那就是GraalVM Native Image。它通过C语言实现了一个超微缩的运行时组件 —— Substrate VM，基本实现了JVM的各种特性，但足够轻量、可以被轻松内嵌，这就让Java语言和工程摆脱JVM的限制，能够真正意义上实现和C/C++一样的AOT编译。这一方案在经过长时间的优化和积累后，已经拥有非常不错的效果，基本上成为Oracle官方首推的Java AOT解决方案。  
Native Image 是一项创新技术，可将 Java 代码编译成独立的本机可执行文件或本机共享库。在构建本机可执行文件期间处理的 Java 字节码包括所有应用程序类、依赖项、第三方依赖库和任何所需的 JDK 类。生成的自包含本机可执行文件特定于不需要 JVM 的每个单独的操作系统和机器体系结构。
### 11.2、演示Native Image构建过程
#### 11.2.1、GraalVM安装
##### （1）下载GraalVM
进入官网下载：[https://www.graalvm.org/downloads/](https://www.graalvm.org/downloads/)
![image-20221207153944132](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048367.png)
![image-20221207152841304](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048924.png)
##### （2）配置环境变量
**添加GRAALVM\_HOME**
![image-20221207110539954](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048933.png)
**把JAVA\_HOME修改为graalvm的位置**
![image-20221207153724340](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048943.png)
**把Path修改位graalvm的bin位置**
![image-20221207153755732](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048946.png)
**使用命令查看是否安装成功**
![image-20221207153642253](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048964.png)
##### （3）安装native-image插件
**使用命令 gu install native-image下载安装**
![image-20221207155009832](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048010.png)
#### 11.2.2、安装C++的编译环境
##### （1）下载Visual Studio安装软件
[https://visualstudio.microsoft.com/zh-hans/downloads/](https://visualstudio.microsoft.com/zh-hans/downloads/)
![image-20221219112426052](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048496.png)
##### （2）安装Visual Studio
![image-20221207155726572](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048493.png)
![image-20221207155756512](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048516.png)
##### （3）添加Visual Studio环境变量
配置INCLUDE、LIB和Path
![image-20221207110947997](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048521.png)
![image-20221207111012582](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048535.png)
![image-20221207111105569](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048545.png)
##### （4）打开工具，在工具中操作
![image-20221207111206279](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048484.png)
#### 11.2.3、编写代码，构建Native Image
##### （1）编写Java代码
```java
public class Hello {
  public static void main(String[] args) {
    System.out.println("hello world");
  }
}
```
##### （2）复制文件到目录，执行编译
![image-20221207111420056](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048482.png)
##### （3）Native Image 进行构建
![image-20221207111509837](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048499.png)
![image-20221207111609878](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048512.png)
##### （4）查看构建的文件
![image-20221207111644950](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048087.png)
##### （5）执行构建的文件
![image-20221207111731150](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048109.png)
可以看到这个Hello最终打包产出的二进制文件大小为11M，这是包含了SVM和JDK各种库后的大小，虽然相比C/C++的二进制文件来说体积偏大，但是对比完整JVM来说，可以说是已经是非常小了。
相比于使用JVM运行，Native Image的速度要快上不少，cpu占用也更低一些，从官方提供的各类实验数据也可以看出Native Image对于启动速度和内存占用带来的提升是非常显著的：
![image-20221207111947283](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048122.png)
![image-20221207112009852](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112048125.png)
> 如果报错如下
![image-20230411221933712](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112219840.png)
`原因，权限不够，以管理员权限启动即可`
> 环境配置
> 
> 如果想像这样添加多行，那么在添加第一行的时候，加一个英文的分号，即 ;
![image-20230411222039492](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112220560.png)
```
INCLUDE
D:\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.35.32215\bin\Hostx64\x64
C:\Program Files (x86)\Windows Kits\10\Include\10.0.22000.0\ucrt
C:\Program Files (x86)\Windows Kits\10\Include\10.0.22000.0\um
C:\Program Files (x86)\Windows Kits\10\Include\10.0.22000.0\shared
```
![image-20230411222158372](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112221435.png)
```
LIB
C:\Program Files (x86)\Windows Kits\10\Include\10.0.22000.0\um\x64
C:\Program Files (x86)\Windows Kits\10\Include\10.0.22000.0\ucrt\64
D:\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.35.32215\bin\Hostx64\x64
```
![image-20230411222305184](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202304112223251.png)
```
path
D:\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.35.32215\bin\Hostx64\x64
```
**java环境变量**
```
从D:\java\jdk改为D:\graalvm-ce-java17-22.3.0
```
