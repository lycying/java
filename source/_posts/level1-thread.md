title: java初级：多线程基础知识
date: 2015-12-01 12:02:19
tags:
categories: [初级]
---

### 进程和线程之间有什么不同？
>一个进程是一个独立(self contained)的运行环境，它可以被看作一个程序或者一个应用。而线程是在进程中执行的一个任务。Java运行环境是一个包含了不同的类和程序的单一进程。线程可以被称为`轻量级进程`。线程需要较少的资源来创建和驻留在进程中，并且可以共享进程中的资源。

### 你是怎么看待线程安全问题的，怎么了解这个概念。
>线程安全问题主要出现在访问临界资源的时候，就是多个线程访问同一个对象的时候，可能会出现无法挽回的损失，特别是在关于资金安全方面的时候，当然还有数据库事务方面的问题。他们很类似，都是要保证数据的原子性。

### 多线程编程的好处是什么？
>在多线程程序中，多个线程被并发的执行以提高程序的效率，CPU不会因为某个线程需要等待资源而进入空闲状态。多个线程共享堆内存(heap memory)，因此创建多个线程去执行一些任务会比创建多个进程更好。

### 有哪些创建线程的方式?
>继承Thread类，覆盖其run方法。
>实现Runnable接口，并作为参数传入到Thread类中。
>使用线程池，提交Runnable或者Callable接口实现。

### 用户线程和守护线程有什么区别？
>当我们在Java程序中创建一个线程，它就被称为用户线程。一个守护线程是在后台执行并且不会阻止JVM终止的线程。当没有用户线程在运行的时候，JVM关闭程序并且退出。一个守护线程创建的子线程依然是守护线程。在Java中，可以调用setDaemon(true)来设置守护线程。

### 什么是线程调度器(Thread Scheduler)和时间分片(Time Slicing)？
>线程调度器是一个操作系统服务，它负责为Runnable状态的线程分配CPU时间。一旦我们创建一个线程并启动它，它的执行便依赖于线程调度器的实现。时间分片是指将可用的CPU时间分配给可用的Runnable线程的过程。分配CPU时间可以基于线程优先级或者线程等待的时间。

### 可重入锁是什么，你认为主要是为了解决什么问题。
>可重入锁的概念是 自己可以再次获得自己的内部锁， 重进入的实现是通过每个锁关联一个请求计数和一个占有它的线程,当计数为0时，认为锁是未被占有的，线程请求一个未被占有的锁时，jvm将记录锁的占有者，并将请求计数置为一，如果同一个线程再次请求，计数器将递增，每次占用线程退出同步块计数器值将递减，直到计数器为0，锁释放。如果内部锁不是可重入的,代码将死锁

### 多线程状态有哪些?
>Java 线程有五种状态创建、就绪、运行、阻塞和死亡。
创建:可以理解我们 new 了一线程对象;
就绪:new 的线程对象调用了 start()方法,但并没有立 即抢到 CPU 时间片;
运行:线程启动后,线程体 run 方法在执行;阻塞:阻塞状 态是指线程因为某些原因放弃 CPU,暂时停止运行。当线程处于阻塞状态时,Java 虚拟机不会给线程分配 CPU,直到线程重新进入就绪状态,它才会有机会获得运行状态;
死亡:当线程执行完 run()方法中的代码或者调用了 stop()方法,又或者 遇到了未捕获的异常,就会退出 run()方法,此时就进入死亡状态,该线程结束 生命周期。

### Thread.getState()会返回哪些值?
>```public enum State {
        NEW,
        RUNNABLE,
        BLOCKED,
        WAITING,
        TIMED_WAITING,
        TERMINATED;
}```
>懂了么？

### THREAD.START()与THREAD.RUN()有什么区别？
>Thread.start()方法(native)启动线程，使之进入就绪状态，当cpu分配时间该线程时，由JVM调度执行run()方法。

### 当synchronized修饰静态方法和普通方法时，有什么差别?
>静态方法是锁类，所有的静态方法都会竞争锁，普通方法是锁对象，所有的普通方法都会竞争锁

### sleep()和 wait()区别
>wait()是 Object 类的方法,在每个类中都可以被调用;而 sleep()是线程类 Thread 中的一个静态方法,无论 New 成多少对象,它都属于调用的类的。
>sleep 方法在同步对象中调用时,会持有对象锁,其它线程必须等待其执行结束,如果时间不到只能调用interrupt()强行打断;在 sleep 时间结束后重新参与cpu时间抢夺,不一定会立刻被执行。
>wait()方法在同步中调用时,会让出对象锁。通常与notify,notifyAll一起使用。

### wait(0)是什么意思？
>就是wait方法，永久等待，直到被唤醒。

### 同步方法和同步代码块的区别是什么？
>在Java语言中，每一个对象有一把锁。线程可以使用synchronized关键字来获取对象上的锁。synchronized关键字可应用在方法级别(粗粒度锁)或者是代码块级别(细粒度锁)。

### Thread类里的join是干什么用的?
>当一个线程调用start后，如果后面的线程想要使用这个线程的执行结果，就需要使用join把并发执行改成`顺序执行`。

### 多次调用Thread的start方法会发生什么情况。
>Thread调用start会给threadStatus设值，所以，重复执行会抛`IllegalThreadStateException`异常。

### 可以直接调用Thread类的run()方法么？
>当然可以，但是如果我们调用了Thread的run()方法，它的行为就会和普通的方法一样，为了在新的线程中执行我们的代码，必须使用Thread.start()方法。

### 如果线程被生成了，但还未被起动，调用它的 join() 方法，什么结果。
>是没有作用的，将直接继续向下执行。

### ThreadLocal是什么?
>线程局部变量高效地为每个使用它的线程提供单独的线程局部变量值的副本。每个线程只能看到与自己相联系的值，而不知道别的线程可能正在使用或修改它们自己的副本。
>举个例子，大家都知道SimpleDateFormat不是线程安全的，在并发环境下，就可以使用ThreadLocal实现。
```
 private ThreadLocal<SimpleDateFormat> sdf = new ThreadLocal() {
        public SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        }
    };
```

### 写一个简单的线程安全的单例
>最简单的:
```public synchronized class Singleton {  
    private static Singleton instance;  
    private Singleton (){}  
    public static Singleton getInstance() {  
        if (instance == null) {  
            instance = new Singleton();  
        }  
        return instance;  
    }  
}  
```
>更详细的请[看这里](http://cantellow.iteye.com/blog/838473)，一般考察到双重锁检查为止。


### 为什么线程通信的方法wait(), notify()和notifyAll()被定义在Object类里？
>Java的每个对象中都有一个锁(monitor，也可以成为监视器) 并且wait()，notify()等方法用于等待对象的锁或者通知其他线程对象的监视器可用。在Java的线程中并没有可供任何对象使用的锁和同步器。这就是为什么这些方法是Object类的一部分，这样Java的每一个类都有用于线程间通信的基本方法

### Atomic类比直接使用传统的java锁机制（阻塞的）有什么好处？
>最大的好处就是可以避免多线程的优先级倒置和死锁情况的发生。

### interrupt、interrupted 、isInterrupted 区别
>[看这里详细介绍](/2015/12/04/interrupt/)
个人总结一下：
- interrupt、interrupted是对象方法，isInterrupted是类方法
- interrupt调用后，线程处于中断状态，中断会抛出interruptedException
- interrupted和isInterrupted得到线程是否处于状态，同时interrupted能清除自己的中断位


### Cached和Fixed线程池的区别是什么
>Cached开辟了一个无限大(Integer.MAX_VALUE)的线程池，Fixed是创建了固定大小的线程池。

### Executors.submit()和Executers.execute()有什么区别?
>submit()接受Runnable和Callable参数，并返回Future
>execute()接受Runnable参数，没有返回值