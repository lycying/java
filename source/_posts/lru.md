title: LRU缓存就像你的鞋柜，附实现攻略
date: 2016-04-07 14:22:03
tags: [arithmetic]
categories: [中级]
---
### 问题
今天我们讲一下怎么实现一个简单的最近最少使用（LRU）的缓存。

### 概念
LRU是Least Recently Used 的缩写，意为“最近最少使用”。
LRU缓存简单的说就是缓存一定量的数据，当超过设定的阈值时就把一些过期的数据删除掉。
举个生活中的例子，你有一堆鞋子，肯定是最新买的最喜欢穿的放在身边，鞋柜满了的话，如果你不是壕，扔鞋子也会先扔破鞋。

如图，把格子想想成你的鞋柜。一个格子只能放一双鞋子哦。
![lru](/lru.png)
- 新数据插入到链表头部；(新鞋子放在最外边)
- 每当缓存命中（即缓存数据被访问），则将数据移到链表头部；(刚穿过的鞋子放在最外面)
- 当链表满的时候，将链表尾部的数据丢弃。(鞋柜满了，就需要忍痛扔破鞋)

当存在热点数据时，LRU的效率很好，但偶发性的、周期性的批量操作会导致LRU命中率急剧下降，缓存污染情况比较严重。

### 实现
Java里面实现LRU缓存通常有多种选择，一种是使用LinkedHashMap，一种是自己设计数据结构，使用链表+HashMap，灵活度更高的可以自定义使用方式。
搞Java的同学真是幸福，因为LinkedHashMap就是一个天生的LRU。
![lru](/happy.gif)

先来看一下LinkedHashMap的构造方法。
```
public LinkedHashMap(int initialCapacity,
float loadFactor,
boolean accessOrder) {
super(initialCapacity, loadFactor);
this.accessOrder = accessOrder;
}
```
- 注意accessOrder这个参数，为true使用访问顺序排序，false使用插入顺序排序。使用访问顺序排序，就是LRU。
- 当需要添加元素时，会调用removeEldestEntry方法，这里就是实现LRU元素过期机制的地方，默认的情况下removeEldestEntry方法只返回false表示元素永远不过期。

根据以上介绍，可以使用继承方式实现一个最简单的LRU.只需要覆盖removeEldestEntry方法。
```
public static class LRULinkedHashMap<K, V> extends LinkedHashMap<K, V> {
        private static final int  LRU_MAX_CAPACITY = 1024;
        public LRULinkedHashMap(int initialCapacity, float loadFactor) {
            super(initialCapacity, loadFactor, true);
        }
        /**
         * @see java.util.LinkedHashMap#removeEldestEntry(java.util.Map.Entry)
         */
        @Override
        protected boolean removeEldestEntry(Entry<K, V> eldest) {
            if(size() > LRU_MAX_CAPACITY) {
                return true;
            }
            return false;
        }
    }
```

### 注意点
使用LRU要注意以下几点:
- 缓存的数据是否真正有热点数据
- 缓存的大小一定要限制，否则会有内存撑爆的危险
- 老生常谈，高并发下一定要注意LRU操作的同步
