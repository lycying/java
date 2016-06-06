title: java中级：多线程
date: 2015-12-01 12:04:44
tags:
categories: [中级]
---
### java.util.concurrent.*, 包，你都用过哪些类?
>Semaphore, CyclicBarrier, CountDownLatch, Lock, ReentrantLock
 Executors, Callable and Future
 Synchronized collections (CopyOnWriteArrayList, ConcurrentHashMap, BlockingQueue)

### 大体描述一下死锁发生的条件。写出一小段死锁的代码。
>死锁产生的四个必要条件。
- 互斥使用，即当资源被一个线程使用(占有)时，别的线程不能使用 
- 不可抢占，资源请求者不能强制从资源占有者手中夺取资源，资源只能由资源占有者主动释放。 
- 请求和保持，即当资源请求者在请求其他的资源的同时保持对原有资源的占有。 
- 循环等待，即存在一个等待队列：P1占有P2的资源，P2占有P3的资源，P3占有P1的资源。这样就形成了一个等待环路。
```
public class DeadlockTest {
    public static void main(String[] args) {   
        String s1 = "S1";
        String s2 = "S2";
        Thread t1 = new Thread(new DeadlockCause(s1, s2));
        Thread t2 = new Thread(new DeadlockCause(s2, s1));
        t1.start();
        t2.start();
        }
    }
    class DeadlockCause implements Runnable {
        private final Object firstLock;
        private final Object secondLock;
        public DeadlockCause(Object o1, Object o2) {
            firstLock = o1;
            secondLock = o2;
        }
        @Override
        public void run() {
            synchronized(firstLock){
                System.out.println(Thread.currentThread().getName() +  " holds the lock on " + firstLock);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException ex) {
                    Logger.getLogger(DeadlockCause.class.getName()).log(Level.SEVERE, null, ex);
                }
                System.out.println(Thread.currentThread().getName() +  " tries to get the lock on " + secondLock);
                synchronized(secondLock){
                    System.out.println(Thread.currentThread().getName() +  " holds the lock on " + secondLock);
                }
            }
        }
    }
```

### 了解volatile么?用它做过什么？
>volatile仅进行内存同步，而不会做线程的同步。监视器锁的happens-before规则保证释放监视器和获取监视器的两个线程之间的内存可见性，这意味着对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。当线程欲引用volatile字段的值时，通常都会发生从主存储器到工作存储器的拷贝操作。而相反地，将值指定给volatile字段后，工作存储器的内容便会映像到主存储器。指定给long、double类型的操作不保证会以atomic的方式来进行。但可以通过volatile来约定用atomic方式来进行赋值操作。
>volatile的典型特点是每次读到的都是最新的值，所以一般最常用的就是控制一个线程的退出（比如tomcat关闭时，要关闭某些线程)
```java
private volatile boolean close = false;
...
run(){while(!close){`do some things`}}
```

### 用java实现一个简单的阻塞队列
>```
 	public BlockingQueue(int limit) {
        this.limit = limit;
    }
    public synchronized void enqueue(Object item) throws InterruptedException {
        while (this.queue.size() == this.limit) {
            wait();
        }
        if (this.queue.size() == 0) {
            notifyAll();
        }
        this.queue.add(item);
    }
    public synchronized Object dequeue() throws InterruptedException {
        while (this.queue.size() == 0) {
            wait();
        }
        if (this.queue.size() == this.limit) {
            notifyAll();
        }
        return this.queue.remove(0);
    }
```

### Java线程池都有哪些参数
>```
public ThreadPoolExecutor(int corePoolSize,
           int maximumPoolSize,
           long keepAliveTime,
           TimeUnit unit,
           BlockingQueue<Runnable> workQueue,
           ThreadFactory threadFactory,
           RejectedExecutionHandler handler) 
```

### 在Java中CycliBarriar和CountdownLatch有什么区别？
>这个线程问题主要用来检测你是否熟悉JDK5中的并发包。这两个的区别是CyclicBarrier可以重复使用已经通过的障碍，而CountdownLatch不能重复使用。

### 有没有接触过虚假唤醒问题
>虚假唤醒就是一些obj.wait()会在除了obj.notify()和obj.notifyAll()的其他情况被唤醒，而此时是不应该唤醒的。解决的办法是基于while来反复判断进入正常操作的临界条件是否满足：
>```   
 synchronized (obj) {  
            while (<condition does not hold>)  
                obj.wait();  
            ... // Perform action appropriate to condition  
        }
```

### 怎么检测wait是正常返回还是因为超时而返回。
>目前没有方法，除非修改jdk源码。

### 谈一谈java-线程池的饱和策略
>ThreadPoolExecutor的饱和策略通过调用setRejectedExecutionHandler来修改。不同的饱和策略如下 ：
>AbortPolicy:中止,executor抛出未检查RejectedExecutionException，调用者捕获这个异常，然后自己编写能满足自己需求的处理代码
>DiscardRunsPolicy:遗弃最旧的，选择丢弃的任务，是本应接下来就执行的任务。
>DiscardPolicy:遗弃会默认放弃最新提交的任务（这个任务不能进入队列等待执行时）
>CallerRunsPolicy：调用者运行，既不会丢弃哪个任务，也不会抛出任何异常，把一些任务推回到调用者那里，以此减缓新任务流。它不会在池线程中执行最新提交的任务，但它会在一个调用了execute的线程中执行。


### 有3个线程ABC。按照ABC来运行（A线程输出A，B线程输出B，C线程输出C，以此类推，循环输出）
>TODO

### 什么是Java线程转储(Thread Dump)，如何得到它？
>线程转储是一个JVM活动线程的列表，它对于分析系统瓶颈和死锁非常有用。有很多方法可以获取线程转储——使用Profiler，`Kill -3`命令，`jstack`工具等等。我更喜欢jstack工具，因为它容易使用并且是JDK自带的。由于它是一个基于终端的工具，所以我们可以编写一些脚本去定时的产生线程转储以待分析。

### Java中的Lock锁，有什么优点?
>Lock接口比同步方法和同步块提供了更具扩展性的锁操作。他们允许更灵活的结构，可以具有完全不同的性质，并且可以支持多个相关类的条件对象。
它的优势有：
- 可以使锁更公平
- 可以使线程在等待锁的时候响应中断
- 可以让线程尝试获取锁，并在无法获取锁的时候立即返回或者等待一段时间
- 可以在不同的范围，以不同的顺序获取和释放锁

### 什么是Callable和Future?
>Callable接口使用泛型去定义它的返回类型。Future提供了get()方法让我们可以等待Callable结束并获取它的执行结果。

### java.util.concurrent.Atomic类中有方法：compareAndSwap和weakCompareAndSwap什么区别?
>JSR规范中说：以原子方式读取和有条件地写入变量但不 创建任何 happen-before 排序，因此不提供与除 weakCompareAndSet 目标外任何变量以前或后续读取或写入操作有关的任何保证。大意就是说调用weakCompareAndSet时并不能保证不存在happen-before的发生（也就是可能存在指令重排序导致此操作失败）。但是从Java源码来看，其实此方法并没有实现JSR规范的要求，最后效果和compareAndSet是`等效的`，都调用了`unsafe.compareAndSwapInt()`完成操作。

### 什么叫spin lock
>CAS

### 什么是互斥锁，什么是读写锁?
>TODO

### java 为什么wait(),notify(),notifyAll()必须在同步方法/代码块中调用？
>[看这里看这里看这里](/2015/12/05/java-wait-notify-sync/)

### 为什么AtomicInteger里面的compareAndSet和weakCompareAndSet方法实现完全一样，注释说明却不同？
>扯淡的问题，如果不想回答，直接说不知道。
>这个问题，一个老外的回答比较靠谱，这里给出链接：[the difference between compareAndSet and weakCompareAndSet](http://stackoverflow.com/questions/2443239/java-atomicinteger-what-are-the-differences-between-compareandset-and-weakcompar)。核心观点就是人家保留更改这个实现的权利。现在一样可能是暂时的，将来可能会不一样，所以使用接口时，还是按照人家接口说明来吧。
