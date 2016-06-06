title: java内置锁的可重入性
date: 2015-12-10 14:40:25
tags:
---
<div class="tip">
锁作为并发共享数据，保证一致性的工具，在JAVA平台有多种实现(如 synchronized 和 ReentrantLock等等) 。在JAVA环境下 ReentrantLock 和synchronized 都是可重入锁。
</div>
我们来看看synchronized,它拥有强制原子性的内置锁机制,是一个重入锁,所以在使用synchronized时,当一个线程请求得到一个对象锁后再次请求此对象锁,可以再次得到该对象锁,就是说在一个synchronized方法/块的内部调用本类的其他synchronized方法/块时，是永远可以拿到锁,如下:
```
public class Child extends Father {
    public static void main(String[] args) {
        Child child = new Child();
        child.doSomething();
    }
    public synchronized void doSomething() {
        System.out.println("child.doSomething()");
        doAnotherThing(); // 调用自己类中其他的synchronized方法
         
    }
    private synchronized void doAnotherThing() {
        super.doSomething(); // 调用父类的synchronized方法
        System.out.println("child.doAnotherThing()");
    }
}
class Father {
    public synchronized void doSomething() {
        System.out.println("father.doSomething()");
    }
}
```
运行结果：
```
child.doSomething()
father.doSomething()
child.doAnotherThing()
```

这里的对象锁只有一个,就是child对象的锁,当执行child.doSomething时，该线程获得child对象的锁，在doSomething方法内执行doAnotherThing时再次请求child对象的锁，因为synchronized是重入锁，所以可以得到该锁，继续在doAnotherThing里执行父类的doSomething方法时第三次请求child对象的锁，同理可得到，如果不是重入锁的话，那这后面这两次请求锁将会被一直阻塞，从而导致死锁。

所以在java内部，同一线程在调用自己类中其他synchronized方法/块或调用父类的synchronized方法/块都不会阻碍该线程的执行，就是说同一线程对同一个对象锁是可重入的，而且同一个线程可以获取同一把锁多次，也就是可以多次重入。**因为java线程是基于“每线程（per-thread）”，而不是基于“每调用（per-invocation）”的**（java中线程获得对象锁的操作是以每线程为粒度的，per-invocation互斥体获得对象锁的操作是以每调用作为粒度的）。


同理，顾名思义，ReentrantLock也是可重入的，如：
```
public class Test implements Runnable {
	ReentrantLock lock = new ReentrantLock();
	public void get() {
		lock.lock();
		System.out.println(Thread.currentThread().getId());
		set();
		lock.unlock();
	}
	public void set() {
		lock.lock();
		System.out.println(Thread.currentThread().getId());
		lock.unlock();
	}
	@Override
	public void run() {
		get();
	}
	public static void main(String[] args) {
		Test ss = new Test();
		new Thread(ss).start();
		new Thread(ss).start();
		new Thread(ss).start();
	}
}
```

