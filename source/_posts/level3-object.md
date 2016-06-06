title: java高级：基础
date: 2015-12-01 12:06:34
tags:
categories: [高级]
---

### 为什么list!=null && !list.isEmpty()效率高于list!=null && list.size()>0 ?
>题目的问法其实有问题，大多数情况下并不是这样的结果。看[这里](http://stackoverflow.com/questions/1508975/why-is-list-size0-slower-than-list-isempty-in-java)，主要考察类加载问题和API设计问题。

### Java的Object既然已经有了`clone()`方法，为什么还要定义一个Cloneable空接口呢？
>首先：如果不继承自Cloneable接口，当调用clone()时会抛出CloneNotSupportedException异常。其实这个接口仅仅是一个标志，而且这个标志也仅仅是针对Object类中clone()方法的。Java内会经常有类似的设计，比如Serializable，使用接口给予某个对象特殊的含义，而它本身什么都不干。

### Java里的类是怎么保存的。Object的头部都有什么。
>TODO

### SLAB是什么?

### AKKA是什么？

### 为什么存储密码用字符数组比字符串更合适？
>因为字符串是不可变对象，它会一直存在内存中直至被垃圾收集器回收。这样有很大的机会长期保留在内存中，会引发安全问题。字符串以普通文本打印在在log文件或控制台中也易引起危险。使用完密码后，要尽快清除它。

### 怎么用双重锁检查来实现一个单例？
>TODO

### 什么是分布式垃圾回收(DGC)？它是如何工作的？
>DGC叫做分布式垃圾回收。RMI使用DGC来做自动垃圾回收。因为RMI包含了跨虚拟机的远程对象的引用，垃圾回收是很困难的。DGC使用引用计数算法来给远程对象提供自动内存管理。

