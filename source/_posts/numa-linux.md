title: (资源)SWAP的罪与罚
date: 2016-07-08 14:24:10
tags: [linux]
categories: [linux]
---

说个案例：一台Apache服务器，由于其MaxClients参数设置过大，并且恰好又碰到访问量激增，结果内存被耗光，从而引发SWAP，进而负载攀升，最终导致宕机。
正所谓：SWAP，性能之大事，死生之地，存亡之道，不可不察也。

## 哪些工具可以监测SWAP
最容易想到的就是free命令了，它指明了当前SWAP的使用情况：
```bash
shell> free -m
             total       used       free
Swap:        34175      11374      22801
```
另一个常用的是sar命令，它能列出系统在各个时间的SWAP使用情况：
```bash
shell> sar -r
kbswpfree kbswpused  %swpused  kbswpcad
 23345644  11650572     33.29   4656908
 23346452  11649764     33.29   4656216
 23346556  11649660     33.29   4650308
 23346932  11649284     33.29   4649888
 23346992  11649224     33.29   4648848
```
不过free命令和sar命令显示的都不是实时数据，如果需要，可以使用vmstat命令：
```bash
shell> vmstat 1
-----------memory------------- ---swap--
  swpd   free   buff   cache     si   so
11647532 123664 305064 7193168    0    0
11647532 123672 305064 7193172    0    0
11647532 125728 305064 7193468    0    0
11647532 125376 305064 7193476    0    0
11647532 124508 305068 7193624    0    0
```
每秒刷新一次结果，在SWAP一栏里列出了相关数据，至于si和so的解释，大致如下：

- si: Amount of memory swapped in from disk (/s).
- so: Amount of memory swapped to disk (/s).

如果它们一直是零当然最好不过了，偶尔不为零也没啥，糟糕的是一直不为零。
前面介绍的方法，看到的都是SWAP的整体情况，可是如果我想查看到底是哪些进程使用了SWAP，应该如何操作呢？这个问题有点棘手，我们来研究一下：
好消息是top命令能提供这个信息，不过缺省并没有显示，我们需要激活一下：

- 打开top；
- 按「f」进入选择字段的界面；
- 按「p」选择「SWAP」字段；
- 按回车确认。

坏消息是top命令提供的SWAP信息只是一个理论值，或者更直白一点儿来说它根本就是不可信的（在top里SWAP的计算公式是：SWAP=VIRT-RES）。
BTW：相比之下，top里的「nFLT」字段更有价值，它表示PageFault的次数。
那到底我们能不能获取到进程的SWAP情况呢？别着急，看代码：
```bash
#!/bin/bash

cd /proc

for pid in [0-9]*; do
    command=$(cat /proc/$pid/cmdline)

    swap=$(
        awk '
            BEGIN  { total = 0 }
            /Swap/ { total += $2 }
            END    { print total }
        ' /proc/$pid/smaps
    )

    if (( $swap > 0 )); then
        if [[ "${head}" != "yes" ]]; then
            echo -e "PID\tSWAP\tCOMMAND"
            head="yes"
        fi

        echo -e "${pid}\t${swap}\t${command}"
    fi
done
```
说明：请使用root权限来运行此脚本。

## 哪些因素可能影响SWAP

内存不足无疑会SWAP，但有些时候，即便看上去内存很充裕，还可能会SWAP，这种现象被称为SWAP Insanity，罪魁祸首主要有以下几点：

Swappiness的迷失

实际上，当可用内存不足时，系统有两个选择：一个是通过SWAP来释放内存，另一个是删除Cache中的Page来释放内存。一个很常见的例子是：当拷贝大文件的时候，时常会发生SWAP现象。这是因为拷贝文件的时候，系统会把文件内容在Cache中按Page来缓存，此时一旦可用内存不足，系统便会倾向于通过SWAP来释放内存。

内核中的swappiness参数可以用来控制这种行为，缺省情况下，swappiness的值是60：

```bash
shell> sysctl -a | grep swappiness
vm.swappiness = 60
```

它的含义是：如果系统需要内存，有百分之六十的概率执行SWAP。知道了这一点，我们很自然的会想到用下面的方法来降低执行SWAP的概率：
```bash
shell> echo "vm.swappiness = 1" >> /etc/sysctl.conf
shell> sysctl -p
这样做的确可以降低执行SWAP的概率，但并不意味着永远不会执行SWAP。据网友报道某些情况下，直接改为0有可能出现灵异问题，所以建议改为1。
```

NUMA的诅咒

NUMA在MySQL社区有很多讨论，这里不多说了，直击NUMA和SWAP的恩怨纠葛。

大概了解一下NUMA最核心的numactl命令：

```bash
shell> numactl --hardware
available: 2 nodes (0-1)
node 0 size: 16131 MB
node 0 free: 100 MB
node 1 size: 16160 MB
node 1 free: 10 MB
node distances:
node   0   1
  0:  10  20
  1:  20  10
```
可以看到系统有两个节点（其实就是两个物理CPU），它们各自分了16G内存，其中零号节点还剩100M内存，一号节点还剩10M内存。设想启动了一个需要11M内存的进程，系统把它分给了一号节点来执行，此时虽然系统总体的可用内存大于该进程需要的内存，但因为一号节点本身剩余的可用内存不足，所以仍然可能会触发SWAP行为。

需要说明的一点事，numactl命令中看到的各节点剩余内存中时不包括Cache内存的，如果需要知道，我们可以利用drop_caches参数先释放它：

```bash
shell> sysctl vm.drop_caches=1
注：这步操作可能会引起系统负载的震荡。
```

另：如何确定一个进程的节点及内存分配情况？网络上有现成的脚本。
https://raw.githubusercontent.com/jeremycole/blog-files/master/numa-maps-summary.pl

如果要规避NUMA对SWAP的影响，最简单的方法就是在启动进程的时候禁用它：

```bash
shell> numactl --interleave=all ...
```

此外，内核参数zone_reclaim_mode通常也很重要，当某个节点可用内存不足时，如果为0的话，那么系统会倾向于从远程节点分配内存；如果为1的话，那么系统会倾向于从本地节点回收Cache内存。多数时候，Cache对性能很重要，所以0是一个更好的选择。

```bash
shell> echo "vm.zone_reclaim_mode = 0" >> /etc/sysctl.conf
shell> sysctl -p
```

早些年，YouTube曾经被SWAP问题困扰过，他们当时的解决方法很极端：删除SWAP！不得不说这真是艺高人胆大，可惜对芸芸众生的我们而言，这实在是太危险了，因为如此一来，一旦内存耗尽，由于没有SWAP的缓冲，系统会立即开始OOM，结果可能会让问题变得更加复杂，所以大家还是安分守己做个老实人吧。
