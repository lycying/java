title: java的long，天生的陷阱专家
date: 2016-04-06 16:54:35
tags: [object,thread]
categories: [中级]
---
工作忙，好久没更新了。有些朋友反映有些东西太难了些。这次来段小程序解解压。

话说，有这么一个小类A。简单的就像嘘了一泡。那么请问在并发的情况下，会有神马问题，会打印Error么。
### 上代码
```
public class A{
    private long a = 0;
    public void set0() {
        a = 0;
    }
    public void set1() {
        a = -1;
    }
    public void check() {
        if (0 != a && -1 != a) {
            System.err.println("Error");
        }
    }
}
```
宋仲基哥哥，你快来，告诉我怎么可能打印Error。
那我们先上一段测试程序。
### 上测试
```
public static void main(final String[] args) {
        final A v = new A();
        final Thread t1 = new Thread() {
            public void run() {
                while (true) {
                    v.set0();
                }
            };
        };
        t1.start();
        final Thread t2 = new Thread() {
            public void run() {
                while (true) {
                    v.set1();
                }
            };
        };
        t2.start();
        final Thread t3 = new Thread() {
            public void run() {
                while (true) {
                    v.check();
                }
            };
        };
        t3.start();
    }
```
实际测试证明，Error出来了，囧。
一个中级的程序员绝对不应该犯如此低级的错误。

### 上原因
问题出在两个地方

- 第一，if判断不是原子的，有可能0!=a通过后，被另外一个线程设置成了0，此时-1!=a 也成立,就囧了
- 第二，在32位机器上，一些赋值操作不是原子性的。在处理大于32位操作时需要处理两次。

看看java的Primitive Types介绍，木有long和double.
http://docs.oracle.com/javase/specs/jls/se7/html/jls-4.html#jls-4.2

concurrent包里面，有不少原子类，如AtomicInteger，AtomicLong等。在设计并发的代码中，我们要尽量的使用这些封装好的原子操作。更下流的做法，使用lock或者synchronized保证操作的原子性。

### 题外话
32位的老古董机器还有不少，32位机器本身只支持4GB的内存，去掉其他资源占用，能留给java进程的也就2G左右，what a pity！
那32的机器有什么优势呢，有些公司还喜欢把64位的大内存宿主机虚拟化成n个32位的小VM，来增加资源的利用率，尤其是docker部署的机器。主要原因是64位的jvm内部的指针占用的是64位（更宽的寻址），相比起来，占用了更多的内存，大约为32位的1.5倍。
要想在64位的jvm上节省更多的内存，可以使用 -XX:+UseCompressedOops，参数开启指针压缩，会节省不少内存。在“一定条件下”，会自动开启此指令。

### END
扯远了，早点休息，麻烦关下灯 我要和宋仲基睡了
![javaobject](/szj.jpg)
