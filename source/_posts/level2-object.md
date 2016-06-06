title: java中级：基础知识
date: 2015-12-01 12:04:33
tags:
categories: [中级]
---
### 什么时候用assert?
>assertion(断言)在软件开发中是一种常用的调试方式，很多开发语言中都支持这种机制。在实现中，assertion就是在程序中的一条语句，它对一个boolean表达式进行检查，一个正确程序必须保证这个boolean表达式的值为true；如果该值为false，说明程序已经处于不正确的状态下，系统将给出警告或退出。一般来说，assertion用于保证程序最基本、关键的正确性。assertion检查通常在开发和测试时开启。为了提高性能，在软件发布后，assertion检查通常是关闭的。



### Java怎么进行深度复制?
> 最简单的方式是使用序列化和反序列化。
	```java
	ByteArrayOutoutStream bo=new ByteArrayOutputStream(); 
	ObjectOutputStream o=new ObjectOutputStream(bo); 
	o.writeObject(this); 
	ByteArrayInputStream bi=new ByteArrayInputStream(bo.toByteArray()); 
	ObjectInputStream oi=new ObjectInputStream(bi); 
	```
>另外可以覆盖Object类的clone方法，记的一定要实现Cloneable接口。
- 在派生类中覆盖基类的clone()方法，并声明为public
- 在派生类的clone()方法中，调用super.clone() [为什么](/2015/12/01/java-clone/)
- 在派生类中实现Cloneable接口

### 枚举可以序列化吗?
>可以

### 怎么同步一个List，有没有线程安全的List？
>要求能够联想到CopyOnWriteList，并能准确描述它的实现。

### ConcurrentHashMap的实现方式是什么？和HashMap比较起来呢？
>TODO

### ConcurrentLinkedQueue是做什么用的？有哪些特性？
>TODO

### 怎么把毫秒转换成秒?Java有没有相应的API支持?
>TimeUnit

### 怎么修改一个方法的可见性，比如把调用某个类的private方法

### 如果一个序列化对象中包含一个非序列化字段，保存时会发生什么？该怎么解决？
>序列化此对象会抛出`NotSerializableException`，给这个字段加上`transient`关键字可以解决。

### 什么是Fail-Fast，什么是Faile-Safe
>Iterator的安全失败是基于对底层集合做拷贝，因此，它不受源集合上修改的影响。java.util包下面的所有的集合类都是快速失败的，而java.util.concurrent包下面的所有的类都是安全失败的。快速失败的迭代器会抛出ConcurrentModificationException异常，而安全失败的迭代器永远不会抛出这样的异常。

### 什么是Java优先级队列(Priority Queue)？
>`PriorityQueue`是一个基于优先级堆的`无界队列`，它的元素是按照自然顺序(natural order)排序的。在创建的时候，我们可以给它提供一个负责给元素排序的比较器。PriorityQueue**不允许null**值，因为他们没有自然顺序，或者说他们没有任何的相关联的比较器。最后，PriorityQueue**不是线程安全的**，入队和出队的时间复杂度是O(log(n))。能够根据它实现**优先级堆**，如最大堆最小堆，也可以很方便的解决`Top K`问题。
