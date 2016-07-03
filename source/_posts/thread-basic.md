title: (转)多线程经典问题
date: 2016-07-03 13:29:27
tags: [thread]
categories: [初级]
---
很多核心面试题来源于多线程(Multi-Threading)和集合框架(Collections Framework)，理解核心线程概念时，娴熟的实际经验是必需的。这篇文章收集了Java线程方面一些典型的问题，这些问题经常被高级工程师所问到。

#### JAVA中多线程同步是什么？
在多线程程序下，同步能控制对共享资源的访问。如果没有同步，当一个Java线程在修改一个共享变量时，另外一个线程正在使用或者更新同一个变量，这样容易导致程序出现错误的结果。

#### 解释实现多线程的几种方法?

一Java线程可以实现Runnable接口或者继承Thread类来实现，当你打算多重继承时，优先选择实现Runnable。

#### THREAD.START()与THREAD.RUN()有什么区别？

Thread.start()方法(native)启动线程，使之进入就绪状态，当cpu分配时间该线程时，由JVM调度执行run()方法。

#### 为什么需要RUN()和START()方法，我们可以只用RUN()方法来完成任务吗？

我们需要run()&start()这两个方法是因为JVM创建一个单独的线程不同于普通方法的调用，所以这项工作由线程的start方法来完成，start由本地方法实现，需要显示地被调用，使用这俩个方法的另外一个好处是任何一个对象都可以作为线程运行，只要实现了Runnable接口，这就避免因继承了Thread类而造成的Java的多继承问题。

#### 什么是THREADLOCAL类，怎么使用它？

ThreadLocal是一个线程级别的局部变量，并非“本地线程”。ThreadLocal为每个使用该变量的线程提供了一个独立的变量副本，每个线程修改副本时不影响其它线程对象的副本(译者注)。

下面是线程局部变量(ThreadLocal variables)的关键点：

一个线程局部变量(ThreadLocal variables)为每个线程方便地提供了一个单独的变量。

ThreadLocal实例通常作为静态的私有的(private static)字段出现在一个类中，这个类用来关联一个线程。

当多个线程访问ThreadLocal实例时，每个线程维护ThreadLocal提供的独立的变量副本。

常用的使用可在DAO模式中见到，当DAO类作为一个单例类时，数据库链接(connection)被每一个线程独立的维护，互不影响。(基于线程的单例)
ThreadLocal难于理解，下面这些引用连接有助于你更好的理解它。

《Good article on ThreadLocal on IBM DeveloperWorks 》、《理解ThreadLocal》、《Managing data : Good example》、《Refer Java API Docs》

#### 什么时候抛出INVALIDMONITORSTATEEXCEPTION异常，为什么？

调用wait()/notify()/notifyAll()中的任何一个方法时，如果当前线程没有获得该对象的锁，那么就会抛出IllegalMonitorStateException的异常(也就是说程序在没有执行对象的任何同步块或者同步方法时，仍然尝试调用wait()/notify()/notifyAll()时)。

由于该异常是RuntimeExcpetion的子类，所以该异常不一定要捕获(尽管你可以捕获只要你愿意).作为RuntimeException，此类异常不会在wait(),notify(),notifyAll()的方法签名提及。

#### SLEEP()、SUSPEND()和WAIT()之间有什么区别？

Thread.sleep()使当前线程在指定的时间处于“非运行”（Not Runnable）状态。线程一直持有对象的监视器。比如一个线程当前在一个同步块或同步方法中，其它线程不能进入该块或方法中。如果另一线程调用了interrupt()方法，它将唤醒那个“睡眠的”线程。

注意：sleep()是一个静态方法。这意味着只对当前线程有效，一个常见的错误是调用t.sleep()，（这里的t是一个不同于当前线程的线程）。即便是执行t.sleep()，也是当前线程进入睡眠，而不是t线程。t.suspend()是过时的方法，使用suspend()导致线程进入停滞状态，该线程会一直持有对象的监视器，suspend()容易引起死锁问题。

object.wait()使当前线程出于“不可运行”状态，和sleep()不同的是wait是object的方法而不是thread。调用object.wait()时，线程先要获取这个对象的对象锁，当前线程必须在锁对象保持同步，把当前线程添加到等待队列中，随后另一线程可以同步同一个对象锁来调用object.notify()，这样将唤醒原来等待中的线程，然后释放该锁。基本上wait()/notify()与sleep()/interrupt()类似，只是前者需要获取对象锁。

#### 在静态方法上使用同步时会发生什么事？

同步静态方法时会获取该类的“Class”对象，所以当一个线程进入同步的静态方法中时，线程监视器获取类本身的对象锁，其它线程不能进入这个类的任何静态同步方法。它不像实例方法，因为多个线程可以同时访问不同实例同步实例方法。

#### 当一个同步方法已经执行，线程能够调用对象上的非同步实例方法吗？

可以，一个非同步方法总是可以被调用而不会有任何问题。实际上，Java没有为非同步方法做任何检查，锁对象仅仅在同步方法或者同步代码块中检查。如果一个方法没有声明为同步，即使你在使用共享数据Java照样会调用，而不会做检查是否安全，所以在这种情况下要特别小心。一个方法是否声明为同步取决于临界区访问(critial section access),如果方法不访问临界区(共享资源或者数据结构)就没必要声明为同步的。

下面有一个示例说明：Common类有两个方法synchronizedMethod1()和method1()，MyThread类在独立的线程中调用这两个方法。

```java
public class Common {

public synchronized void synchronizedMethod1() {
System.out.println(“synchronizedMethod1 called”);
try {
Thread.sleep(1000);
} catch (InterruptedException e) {
e.printStackTrace();
}
System.out.println(“synchronizedMethod1 done”);
}
public void method1() {
System.out.println(“Method 1 called”);
try {
Thread.sleep(1000);
} catch (InterruptedException e) {
e.printStackTrace();
}
System.out.println(“Method 1 done”);
}
}

public class MyThread extends Thread {
private int id = 0;
private Common common;

public MyThread(String name, int no, Common object) {
super(name);
common = object;
id = no;
}

public void run() {
System.out.println(“Running Thread” + this.getName());
try {
if (id == 0) {
common.synchronizedMethod1();
} else {
common.method1();
}
} catch (Exception e) {
e.printStackTrace();
}
}

public static void main(String[] args) {
Common c = new Common();
MyThread t1 = new MyThread(“MyThread-1″, 0, c);
MyThread t2 = new MyThread(“MyThread-2″, 1, c);
t1.start();
t2.start();
}
}
```

这里是程序的输出：

```bash
Running ThreadMyThread-1
synchronizedMethod1 called
Running ThreadMyThread-2
Method 1 called
synchronizedMethod1 done
Method 1 done
```

结果表明即使synchronizedMethod1()方法执行了，method1()也会被调用。

#### 在一个对象上两个线程可以调用两个不同的同步实例方法吗？
不能，因为一个对象已经同步了实例方法，线程获取了对象的对象锁。所以只有执行完该方法释放对象锁后才能执行其它同步方法。
看下面代码示例非常清晰：Common 类 有synchronizedMethod1()和synchronizedMethod2()方法，MyThread调用这两个方法。
```java
public class Common {
public synchronized void synchronizedMethod1() {
System.out.println(“synchronizedMethod1 called”);
try {
Thread.sleep(1000);
} catch (InterruptedException e) {
e.printStackTrace();
}
System.out.println(“synchronizedMethod1 done”);
}

public synchronized void synchronizedMethod2() {
System.out.println(“synchronizedMethod2 called”);
try {
Thread.sleep(1000);
} catch (InterruptedException e) {
e.printStackTrace();
}
System.out.println(“synchronizedMethod2 done”);
}
}

public class MyThread extends Thread {
private int id = 0;
private Common common;

public MyThread(String name, int no, Common object) {
super(name);
common = object;
id = no;
}

public void run() {
System.out.println(“Running Thread” + this.getName());
try {
if (id == 0) {
common.synchronizedMethod1();
} else {
common.synchronizedMethod2();
}
} catch (Exception e) {
e.printStackTrace();
}
}

public static void main(String[] args) {
Common c = new Common();
MyThread t1 = new MyThread(“MyThread-1″, 0, c);
MyThread t2 = new MyThread(“MyThread-2″, 1, c);
t1.start();
t2.start();
}
}
```
#### 什么是死锁
死锁就是两个或两个以上的线程被无限的阻塞，线程之间相互等待所需资源。这种情况可能发生在当两个线程尝试获取其它资源的锁，而每个线程又陷入无限等待其它资源锁的释放，除非一个用户进程被终止。就JavaAPI而言，线程死锁可能发生在一下情况。

- 当两个线程相互调用Thread.join()
- 当两个线程使用嵌套的同步块，一个线程占用了另外一个线程必需的锁，互相等待时被阻塞就有可能出现死锁。

#### 什么是线程饿死，什么是活锁？

线程饿死和活锁虽然不想是死锁一样的常见问题，但是对于并发编程的设计者来说就像一次邂逅一样。
当所有线程阻塞，或者由于需要的资源无效而不能处理，不存在非阻塞线程使资源可用。JavaAPI中线程活锁可能发生在以下情形：
- 当所有线程在程序中执行Object.wait(0)，参数为0的wait方法。程序将发生活锁直到在相应的对象上有线程调用Object.notify()或者Object.notifyAll()。
- 当所有线程卡在无限循环中。

这里的问题并不详尽，我相信还有很多重要的问题并未提及，您认为还有哪些问题应该包括在上面呢？欢迎在评论中分享任何形式的问题与建议。
