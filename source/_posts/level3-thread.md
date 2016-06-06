title: java高级：多线程
date: 2015-12-01 12:06:51
tags:
categories: [高级]
---
### 了解ABA问题么?简单谈下你的理解。
### happen-before规则都有哪些？
>TODO

### 什么是false-sharing，最好举个例子
>TODO

### LockSupport.park()和unpark()，与object.wait()和notify()的区别？ 
>- 面向的主体不一样。LockSuport主要是针对Thread进进行阻塞处理，可以指定阻塞队列的目标对象，每次可以指定具体的线程唤醒。Object.wait()是以对象为纬度，阻塞当前的线程和唤醒单个(随机)或者所有线程。 
- 实现机制不同。虽然LockSuport可以指定monitor的object对象，但和object.wait()，两者的阻塞队列并不交叉。

### LockSupport.park(Object blocker)传递的blocker对象做什么用？ 
>TODO

### park能响应中断请求么?
>因为调用park而阻塞的话，能够响应中断请求(中断状态被设置成true)，但是不会抛出InterruptedException

### 并发和并行是一个概念么？
>TODO
