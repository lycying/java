title: JAVA-CAS简介以及ABA问题
date: 2015-12-02 11:46:37
tags:
---
## CAS简介
`java.util.concurrent`包完全建立在CAS之上的，没有CAS就不会有此包。可见CAS的重要性。 CAS:Compare and Swap, 翻译成比较并交换。 `java.util.concurrent`包中借助CAS实现了区别于synchronouse同步锁的一种`乐观锁`。
CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。
### 举例
拿`AtomicInteger`来说
```
public final int incrementAndGet() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return next;
    }
}
//compareAndSet如下,valueOffset这货就是内存地址
//比较current的值是否和内存里的值相同，如果是，将内存值设置为next
public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

### 语义
CAS原子操作在维基百科中的代码描述如下，也就是检查内存*reg里的值是不是oldval，如果是的话，则对其赋值newval:
```
int compare_and_swap(int* reg, int oldval, int newval){
     ATOMIC();
     int old_reg_val = *reg;
     if (old_reg_val == oldval)
        *reg = newval;
      END_ATOMIC();
      return old_reg_val;
}
```
上面的代码总是返回old_reg_value，调用者如果需要知道是否更新成功还需要做进一步判断，为了方便，它可以变种为直接返回是否更新成功，如下：
```
bool compare_and_swap (int *accum, int *dest, int newval){
      if ( *accum == *dest ) {
          *dest = newval;
          return true;
      }
      return false;
}
```
CAS操作是硬件层面的原语，是原子操作。

## CAS缺点

CAS虽然很高效的解决原子操作，但是CAS仍然存在三大问题。ABA问题，循环时间长开销大和只能保证一个共享变量的原子操作

### ABA问题
因为CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A－B－A 就会变成1A-2B－3A。 从Java1.5开始JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。 关于ABA问题参考文档:(http://blog.hesey.net/2011/09/resolve-aba-by-atomicstampedreference.html)
### 循环时间长开销大
自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。
### 只能保证一个共享变量的原子操作
当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。

## 使用AtomicStampedReference解决ABA问题
各种乐观锁的实现中通常都会用版本戳version来对记录或对象标记，避免并发操作带来的问题，在Java中，AtomicStampedReference<E>也实现了这个作用，它通过包装[E,Integer]的元组来对对象标记版本戳stamp，从而避免ABA问题，例如下面的代码分别用AtomicInteger和AtomicStampedReference来对初始值为100的原子整型变量进行更新，AtomicInteger会成功执行CAS操作，而加上版本戳的AtomicStampedReference对于ABA问题会执行CAS失败：
```
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;
public class ABA {
        private static AtomicInteger atomicInt = new AtomicInteger(100);
        private static AtomicStampedReference atomicStampedRef = new AtomicStampedReference(100, 0);
        public static void main(String[] args) throws InterruptedException {
                Thread intT1 = new Thread(new Runnable() {
                        @Override
                        public void run() {
                                atomicInt.compareAndSet(100, 101);
                                atomicInt.compareAndSet(101, 100);
                        }
                });
                Thread intT2 = new Thread(new Runnable() {
                        @Override
                        public void run() {
                                try {
                                        TimeUnit.SECONDS.sleep(1);
                                } catch (InterruptedException e) {
                                }
                                boolean c3 = atomicInt.compareAndSet(100, 101);
                                System.out.println(c3); // true
                        }
                });
                intT1.start();
                intT2.start();
                intT1.join();
                intT2.join();
                Thread refT1 = new Thread(new Runnable() {
                        @Override
                        public void run() {
                                try {
                                        TimeUnit.SECONDS.sleep(1);
                                } catch (InterruptedException e) {
                                }
                                atomicStampedRef.compareAndSet(100, 101, atomicStampedRef.getStamp(), atomicStampedRef.getStamp() + 1);
                                atomicStampedRef.compareAndSet(101, 100, atomicStampedRef.getStamp(), atomicStampedRef.getStamp() + 1);
                        }
                });
                Thread refT2 = new Thread(new Runnable() {
                        @Override
                        public void run() {
                                int stamp = atomicStampedRef.getStamp();
                                try {
                                        TimeUnit.SECONDS.sleep(2);
                                } catch (InterruptedException e) {
                                }
                                boolean c3 = atomicStampedRef.compareAndSet(100, 101, stamp, stamp + 1);
                                System.out.println(c3); // false
                        }
                });
                refT1.start();
                refT2.start();
        }
}
```
