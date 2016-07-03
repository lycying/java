title: false-sharing问题(Cache line引起)
date: 2015-12-03 12:03:43
tags: [thread]
categories: [高级]
---
### 简介

#### 什么叫Cache line
>CPU利用cache和内存之间交换数据的最小粒度不是字节，而是称为cacheline的一块固定大小的区域。CPU缓存是一种高效的非链式结构的hash map，每个桶（bucket）通常是64个字节。这就是一个“缓存行（cache line）”。缓存行是内存交换的实际单位。

[查看wiki详细介绍](https://en.wikipedia.org/wiki/CPU_cache#Cache_entry_structure)

### 问题
[Avoiding and Identifying False Sharing Among Threads](https://software.intel.com/en-us/articles/avoiding-and-identifying-false-sharing-among-threads/)

当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享。在多处理器，多线程情况下，如果两个线程分别运行在不同的CPU上，而其中某个线程修改了cache line中的元素，由于cache一致性的原因，另一个线程的cache line被宣告无效，在下一次访问时会出现一次cache line miss。

一句话，这是由于两个线程的`Cache line`有重合引起的。
如图:

![](/false-sharing.gif)

 如图所示，thread1修改了memory灰化区域的第[2]个元素，而Thread0只需要读取灰化区域的第[1]个元素，由于这段memory被载入了各自CPU的硬件cache中，虽然在memory的角度这两种的访问时隔离的，但是由于错误的紧凑地放在了一起，而导致了，thread1的修改，在cache一致性的要求下，宣告了运行Thread0的CPU0的cache line非法，从而出现了一次miss，导致从小从memory中读取到cache line中，而一致性的代价付出了，但结果并不是thread0所care的，导致了效率的降低。

 在Java程序中,数组的成员在缓存中也是连续的. 其实从Java对象的相邻成员变量也会加载到同一缓存行中. 如果多个线程操作不同的成员变量, 但是相同的缓存行, 伪共享(False Sharing)问题就发生了.


### 代码示例
[看一个测试例子](http://mechanical-sympathy.blogspot.com/2011/07/false-sharing.html)

注意这一行，使用了7个long来填充缓存行。
```
public long p1, p2, p3, p4, p5, p6, p7;
```

```
public final class FalseSharing
    implements Runnable{
    public final static int NUM_THREADS = 4; // change
    public final static long ITERATIONS = 500L * 1000L * 1000L;
    private final int arrayIndex;
    private static VolatileLong[] longs = new VolatileLong[NUM_THREADS];
    static {
        for (int i = 0; i < longs.length; i++) {
            longs[i] = new VolatileLong();
        }
    }
    public FalseSharing(final int arrayIndex) {
        this.arrayIndex = arrayIndex;
    }
    public static void main(final String[] args) throws Exception {
        final long start = System.nanoTime();
        runTest();
        System.out.println("duration = " + (System.nanoTime() - start));
    }
    private static void runTest() throws InterruptedException {
        Thread[] threads = new Thread[NUM_THREADS];
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(new FalseSharing(i));
        }
        for (Thread t : threads) {
            t.start();
        }
        for (Thread t : threads) {
            t.join();
        }
    }
    public void run() {
        long i = ITERATIONS + 1;
        while (0 != --i) {
            longs[arrayIndex].value = i;
        }
    }
    public final static class VolatileLong {
        public volatile long value = 0L;
        public long p1, p2, p3, p4, p5, p6, p7; // comment out
    }
}
```

高性能无锁队列[Disruptor](http://lmax-exchange.github.io/disruptor/) 的[RingBuffer.java (查看源码)](https://github.com/LMAX-Exchange/disruptor/blob/master/src/main/java/com/lmax/disruptor/RingBuffer.java) 就使用了这种对齐方式。
