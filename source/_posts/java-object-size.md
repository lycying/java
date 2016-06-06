title: java Object类占用内存大小计算 
date: 2015-12-02 14:53:03
tags:
---
### 前言
这部分信息，对于初级程序员来说并没有实质性的帮助。但对于中高级程序员来说，随着对系统开发的深入，是必须了解的一部分。

前提：JVM规范中没有针对运行时Java对象的内存结构的说明，这也就是说JVM供应商可以按照自己的需要来实现这一点。后果就是，同一个类在不同的JVM上运行的实例对象占用的内存大小会有差别。**假设运行在Sun HotSpot的JVM 32位Java上**

### 一个Object占用的空间
在Java中，一个空Object对象的大小是**8byte**，这个大小只是保存堆中一个没有任何属性的对象的大小。看下面语句：
```
Object ob = new Object();
``` 
以上语句声明了一个新的对象，它所占的空间为：**4byte+8byte**。除了**8byte**的堆内存，还有**4byte**的引用空间大小。



类型  | 大小(byte)
------------- | -------------
byte  	| 1
short  	| 2
int 	| 4
long 	| 8
float 	| 4
double 	| 8
boolean | ?
array	| 12

关于boolean占用的大小，有点特殊。
>虽然 Java 虚拟机定义了 boolean 这种数据类型，但是只对它提供了非常有限的支持。在Java 虚拟机中没有任何供 boolean 值专用的字节码指令，在 Java 语言之中涉及到 `boolean` 类型值的运算，在编译之后都使用 Java 虚拟机中的 int 数据类型来代替。
>-
>Java 虚拟机直接支持 boolean 类型的数组，虚拟机的 `newarray` 指令可以创建这种数组。 boolean 的数组类型的访问与修改共用 byte 类型数组的 `baload` 和 `bastore` 指令。
参考 http://stackoverflow.com/questions/1907318/java-boolean-primitive-type-size


### java Object 存储
更多资料：
[Java对象内存结构](http://www.importnew.com/1305.html)
[HotSpot虚拟机对象探秘](http://www.infoq.com/cn/articles/jvm-hotspot)
#### 结构
我们先来看看在堆中单个的Object长什么样子
![javaobject](/javaobject.png)

#### 对象头(Mark Word)
在Sun JVM中，（除了数组之外的）对象都有两个机器字（words）的头部。第一个字中包含这个对象的标示哈希码以及其他一些类似锁状态和对象分代年龄等标识信息，第二个字中包含一个指向对象的类的引用。
#### 基本类型
接下来实例数据部分是对象真正存储的有效信息，也既是我们在程序代码里面所定义的各种类型的字段内容，无论是从父类继承下来的，还是在子类中定义的。
#### 引用类型
每个引用类型占用 4 bytes
#### 填充物
对齐填充并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。由于HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说就是对象的大小必须是8字节的整数倍。对象头部分正好似8字节的倍数（1倍或者2倍），因此当对象实例数据部分没有对齐的话，就需要通过对齐填充来补全。
### 例子
* 一个空对象（没有声明任何变量）占用 8 bytes -- > 对象头 占用 8 bytes
* 只声明了一个boolean类型变量的类，占用 16 bytes --> 对象头(8 bytes) + boolean (1 bytes) + 填充物（7 bytes）
* 声明了8个boolean类型变量的类，占用 16 bytes --> 对象头(8 bytes) + boolean (1 bytes) * 8
