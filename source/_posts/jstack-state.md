title: jstack线程dump输出状态解释
date: 2015-12-09 17:17:26
tags: [shell,tools]
categories: [中级]
---
执行`jstack`命令，将得到进程的堆栈信息。我一般使用`jstack -l pid`来得到长列表，显示其详细信息。
有时线程挂起的时候，需要执行`jstack -F pid`来获取。
<div class="tip">
在实际运行中，往往一次 dump的信息，还不足以确认问题。建议产生三次 dump信息，如果每次 dump都指向同一个问题，我们才确定问题的典型性。

堆栈信息只是一种参考，一些正常RUNNING的线程，由于复杂网络环境和IO的影响，也是有问题的，用jstack就无法定位，需要结合对业务代码的理解。
</div>

### 线程的状态分析
线程的定义里，有状态：
```java
public enum State {
        NEW,
        RUNNABLE,
        BLOCKED,
        WAITING,
        TIMED_WAITING,
        TERMINATED;
}```

在dump 文件里，写法可能不太一样：
- 死锁，Deadlock（重点关注）
- 执行中，Runnable   
- 等待资源，Waiting on condition（重点关注）
- 等待获取监视器，Waiting on monitor entry（重点关注）
- 对象等待中，Object.wait() 或 TIMED_WAITING
- 暂停，Suspended
- 阻塞，Blocked（重点关注）  
- 停止，Parked

### Runnable
线程正在运行中。一般指该线程正在执行状态中，该线程占用了资源，正在处理某个请求，有可能正在传递SQL到数据库执行，有可能在对某个文件操作，有可能进行数据类型等转换。

### Deadlock
```
"t2" prio=6 tid=0x02bcf000 nid=0xc70 waiting for monitor entry [0x02f6f000]  
   java.lang.Thread.State: BLOCKED (on object monitor)  
    at com.demo.DeadLock$2.run(DeadLock.java:40)  
    - waiting to lock <0x22a297a8> (a java.lang.Object)  
    - locked <0x22a297b0> (a java.lang.Object)  
   Locked ownable synchronizers:  
    - None    
"t1" prio=6 tid=0x02bce400 nid=0xba0 waiting for monitor entry [0x02f1f000]  
   java.lang.Thread.State: BLOCKED (on object monitor)  
    at com.demo.DeadLock$1.run(DeadLock.java:25)  
    - waiting to lock <0x22a297b0> (a java.lang.Object)  
    - locked <0x22a297a8> (a java.lang.Object)  
   Locked ownable synchronizers:  
    - None  
```
上面是一个典型的死锁堆栈，t1线程lock在地址`0x22a297a8`，同时t2线程在`waiting to lock`这个地址。更有意思的是，堆栈也记录了发生死锁的代码行数，这对我们定位问题起到很大的帮助。

### Wait on condition
**等待资源，或等待某个条件的发生**。具体原因需结合 stacktrace来分析。

- 常见情况是该线程在 sleep，等待 sleep的时间到了时候，将被唤醒。关键字：`TIMED_WAITING`,`sleeping`,`parking`。TIMED_WAITING可能是调用了有超时参数的wait所引起的。parking指线程处于挂起中。
下面是一个典型的sleep引起的`Wait on condition`。
 ```
 "thread-1" prio=10 tid=0x00007fbe985cd000 nid=0x7bc6 waiting on condition [0x00007fbe65848000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at com.xxx.MonitorManager$2.run(MonitorManager.java:124)
        at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:895)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:918)
        at java.lang.Thread.run(Thread.java:662)
```
- 如果发现有大量的线程都在处在 Wait on condition，从线程 stack看，正等待网络读写，这可能是一个网络瓶颈的征兆。因为网络阻塞导致线程无法执行。一种情况是网络非常忙，几乎消耗了所有的带宽，仍然有大量数据等待网络读 写；另一种情况也可能是网络空闲，但由于路由等问题，导致包无法正常的到达。可以结合其他网络分析工具定位问题。如下面的堆栈，线程等待在LinkedBlockingQueue上等待数据的生成。
- 如果堆栈信息明确是应用代码，则证明该线程正在等待资源。一般是大量读取某资源，且该资源采用了资源锁的情况下，线程进入等待状态，等待资源的读取。
```
"IoWaitThread" prio=6 tid=0x0000000007334800 nid=0x2b3c waiting on condition [0x000000000893f000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000007d5c45850> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:156)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:1987)
        at java.util.concurrent.LinkedBlockingDeque.takeFirst(LinkedBlockingDeque.java:440)
        at java.util.concurrent.LinkedBlockingDeque.take(LinkedBlockingDeque.java:629)
        at com.xxx.ThreadIoWaitState$IoWaitHandler2.run(ThreadIoWaitState.java:89)
        at java.lang.Thread.run(Thread.java:662)
```

### Waiting on monitor entry和Object.wait()
**意味着线程在等待进入一个临界区**
Monitor是 Java中用以实现线程之间的互斥与协作的主要手段，它可以看成是对象或者 Class的锁。每一个对象都有，也仅有一个 monitor。
![](/java-entry-set-wait-set.png)
- 所有期待获得锁的线程，在锁已经被其它线程拥有的时候，这些期待获得锁的线程就进入了`Object Lock`的`entry set`区域。
- 所有曾经获得过锁，但是由于其它必要条件不满足而需要wait的时候，线程就进入了`Object Lock`的`wait set`区域 。
- 在`wait set`区域的线程获得`notify/notifyAll`通知的时候，随机的一个Thread（notify）或者是全部的Thread（notifyALL）从Object Lock的wait set区域进入了entry set中。
- 在当前拥有锁的线程释放掉锁的时候，处于该`Object Lock`的`entryset`区域的线程都会抢占该锁，但是只能有任意的一个Thread能取得该锁，而其他线程依然在`entry set`中等待下次来抢占到锁之后再执行。

看下面的堆栈，线程在等待数据库连接池返回一个可用的链接
```
" DB-Processor-13" daemon prio=5 tid=0x003edf98 nid=0xca waiting for monitor entry [0x000000000825f000]
java.lang.Thread.State: BLOCKED (on object monitor)
       at beans.ConnectionPool.getConnection(ConnectionPool.java:102)
       - waiting to lock <0xe0375410> (a beans.ConnectionPool)
       at xxx.getTodayCount(ServiceCnt.java:111)
       at xxx.ServiceCnt.insertCount(ServiceCnt.java:43)
"DB-Processor-14" daemon prio=5 tid=0x003edf98 nid=0xca waiting for monitor entry [0x000000000825f020]
java.lang.Thread.State: BLOCKED (on object monitor)
       at beans.ConnectionPool.getConnection(ConnectionPool.java:102)
       - waiting to lock <0xe0375410> (a beans.ConnectionPool)
       at xxx.ServiceCnt.getTodayCount(ServiceCnt.java:111)
       at xxx.ServiceCnt.insertCount(ServiceCnt.java:43)
" DB-Processor-3" daemon prio=5 tid=0x00928248 nid=0x8b waiting for monitor entry [0x000000000825d080]
java.lang.Thread.State: RUNNABLE
       at oracle.jdbc.driver.OracleConnection.isClosed(OracleConnection.java:570)
       - waiting to lock <0xe03ba2e0> (a oracle.jdbc.driver.OracleConnection)
       at beans.ConnectionPool.getConnection(ConnectionPool.java:112)
       - locked <0xe0386580> (a java.util.Vector)
       - locked <0xe0375410> (a beans.ConnectionPool)
       at xxx.Cue_1700c.GetNationList(Cue_1700c.java:66)
       at org.apache.jsp.cue_1700c_jsp._jspService(cue_1700c_jsp.java:120)
```

### Blocked
线程阻塞，是指当前线程执行过程中，所需要的资源长时间等待却一直未能获取到，被容器的线程管理器标识为阻塞状态，可以理解为等待资源超时的线程。
如果线程处于Blocked状态，但是原因不清楚。可以使用`jstack -m pid`得到线程的mixed信息。
```
----------------- t@13 -----------------
0xff31e8b8      ___lwp_cond_wait + 0x4
0xfea8c810      void ObjectMonitor::EnterI(Thread*) + 0x2b8
0xfeac86b8      void ObjectMonitor::enter2(Thread*) + 0x250
```
例如，以上信息表明，线程在尝试进入同步块时阻塞了。

### 关于concurrent的Lock
在JDK 5.0中，引入了`Lock`机制，从而使开发者能更灵活的开发高性能的并发多线程程序，可以替代以往JDK中的`synchronized`和`Monitor`的机制。但是，要注意的是，因为Lock类只是一个普通类，JVM无从得知Lock对象的占用情况，所以在线程DUMP中，也不会包含关于 Lock的信息，关于死锁等问题，就不如用 synchronized的编程方式容易识别。

### jstack解决CPU过高的问题
>第一步，找到占用cpu最高的一个线程
>方法一：`top -p [pid]`
>方法二：`ps -mo spid,lwp,stime,time,%cpu -p [pid]`
>方法三：直接`top`,然后`shift+h`
>第二步，将其转化成16进制。假使我们得到的线程号为n，接下来将它转成16进制，记为spid
>方法一：`echo "obase=64;n"|bc `
>方法二：`printf 0x%x n`
>下一步，执行`jstack -l  pid| grep spid -A 100` 打印后面100行分析问题

下面便是一个查找的结果
```
"http-8081-11" daemon prio=10 tid=0x00002aab049a1800 nid=0x52f1 in Object.wait() [0x0000000042c75000]  
   java.lang.Thread.State: WAITING (on object monitor)  
     at java.lang.Object.wait(Native Method)  
     at java.lang.Object.wait(Object.java:485)  
     at org.apache.tomcat.util.net.JIoEndpoint$Worker.await(JIoEndpoint.java:416)  
```
