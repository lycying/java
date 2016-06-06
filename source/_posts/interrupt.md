title: interrupt、interrupted 、isInterrupted 区别
date: 2015-12-04 17:04:39
tags:
---
[个人认为这个介绍很不错,转了](http://m.oschina.net/blog/121933)

### interrupt 
`interrupt`方法用于中断线程。调用该方法的线程的状态为将被置为"中断"状态。
注意：线程中断仅仅是置线程的中断状态位，不会停止线程。需要用户自己去监视线程的状态为并做处理。支持线程中断的方法（也就是线程中断后会抛出`interruptedException`的方法）就是在监视线程的中断状态，一旦线程的中断状态被置为“中断状态”，就会抛出中断异常。


### interrupted 和 isInterrupted

首先看一下该方法的实现：
```
public static boolean interrupted () {
     return currentThread().isInterrupted(true);
}
```
该方法就是直接调用当前线程的`isInterrupted(true)`方法。
然后再来看一下 isInterrupted的实现：
```
public boolean isInterrupted () {
     return isInterrupted( false);
}
```
这两个方法有两个主要区别：

- interrupted 是作用于当前线程，isInterrupted 是作用于调用该方法的线程对象所对应的线程。（线程对象对应的线程不一定是当前运行的线程。例如我们可以在A线程中去调用B线程对象的isInterrupted方法。）
这两个方法最终都会调用同一个方法，只不过参数一个是true，一个是false；

- 第二个区别主要体现在调用的方法的参数上，让我们来看一看这个参数是什么含义
先来看一看被调用的方法 `isInterrupted(boolean arg)`的定义：
```
 private native boolean isInterrupted( boolean ClearInterrupted);
```
原来这是一个本地方法，看不到源码。不过没关系，通过参数名我们就能知道，这个参数代表是否要清除状态位。
如果这个参数为true，说明返回线程的状态位后，要清掉原来的状态位（恢复成原来情况）。这个参数为false，就是直接返回线程的状态位。

这两个方法很好区分，**只有当前线程才能清除自己的中断位**（对应interrupted（）方法）

