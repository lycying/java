title: jstat详解
date: 2015-12-02 11:31:31
tags:
---
#### jstat -gc pid
可以显示gc的信息，查看gc的次数，及时间。
其中最后五项，分别是young gc的次数，young gc的时间，full gc的次数，full gc的时间，gc的总时间。
#### jstat -gccapacity pid
可以显示，VM内存中三代（young,old,perm）对象的使用和占用大小，
如：PGCMN显示的是最小perm的内存使用量，PGCMX显示的是perm的内存最大使用量，
PGC是当前新生成的perm内存占用量，PC是但前perm内存占用量。
其他的可以根据这个类推， OC是old内纯的占用量。
#### jstat -gcutil pid
统计gc信息统计。
#### jstat -gcnew pid
年轻代对象的信息。
#### jstat -gcnewcapacity pid
年轻代对象的信息及其占用量。
#### jstat -gcold pid
old代对象的信息。
#### jstat -gcoldcapacity pid
old代对象的信息及其占用量。
#### jstat -gcpermcapacity pid
perm对象的信息及其占用量。
#### jstat -class pid
显示加载class的数量，及所占空间等信息。
#### jstat -compiler pid
显示VM实时编译的数量等信息。
#### jstat -printcompilation pid
当前VM执行的信息。

#### 一些术语的中文解释

- `S0C`：年轻代中第一个survivor（幸存区）的容量 (字节)
- `S1C`：年轻代中第二个survivor（幸存区）的容量 (字节)
- `S0U`：年轻代中第一个survivor（幸存区）目前已使用空间 (字节)
- `S1U`：年轻代中第二个survivor（幸存区）目前已使用空间 (字节)
- `EC`：年轻代中Eden（伊甸园）的容量 (字节)
- `EU`：年轻代中Eden（伊甸园）目前已使用空间 (字节)
- `OC`：Old代的容量 (字节)
- `OU`：Old代目前已使用空间 (字节)
- `PC`：Perm(持久代)的容量 (字节)
- `PU`：Perm(持久代)目前已使用空间 (字节)
- `YGC`：从应用程序启动到采样时年轻代中gc次数
- `YGCT`：从应用程序启动到采样时年轻代中gc所用时间(s)
- `FGC`：从应用程序启动到采样时old代(全gc)gc次数
- `FGCT`：从应用程序启动到采样时old代(全gc)gc所用时间(s)
- `GCT`：从应用程序启动到采样时gc用的总时间(s)
- `NGCMN`：年轻代(young)中初始化(最小)的大小 (字节)
- `NGCMX`：年轻代(young)的最大容量 (字节)
- `NGC`：年轻代(young)中当前的容量 (字节)
- `OGCMN`：old代中初始化(最小)的大小 (字节) 
- `OGCMX`：old代的最大容量 (字节)
- `OGC`：old代当前新生成的容量 (字节)
- `PGCMN`：perm代中初始化(最小)的大小 (字节) 
- `PGCMX`：perm代的最大容量 (字节)   
- `PGC`：perm代当前新生成的容量 (字节)
- `S0`：年轻代中第一个survivor（幸存区）已使用的占当前容量百分比
- `S1`：年轻代中第二个survivor（幸存区）已使用的占当前容量百分比
- `E`：年轻代中Eden（伊甸园）已使用的占当前容量百分比
- `O`：old代已使用的占当前容量百分比
- `P`：perm代已使用的占当前容量百分比
- `S0CMX`：年轻代中第一个survivor（幸存区）的最大容量 (字节)
- `S1CMX` ：年轻代中第二个survivor（幸存区）的最大容量 (字节)
- `ECMX`：年轻代中Eden（伊甸园）的最大容量 (字节)
- `DSS`：当前需要survivor（幸存区）的容量 (字节)（Eden区已满）
- `TT`： 持有次数限制
- `MTT`： 最大持有次数限制
