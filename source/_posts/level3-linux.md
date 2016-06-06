title: java高级：linux和shell编程
date: 2015-12-01 12:07:26
tags:
categories: [高级]
---
### 怎么查看某个进程里面占用CPU最高的一个线程？
>第一步，找到占用cpu最高的一个线程
>方法一：`top -p [pid]`
>方法二：`ps -mo spid,lwp,stime,time,%cpu -p [pid]`
>方法三：直接`top`,然后`shift+h`
>第二步，将其转化成16进制。假使我们得到的线程号为n，接下来将它转成16进制，记为spid
>方法一：`echo "obase=64;n"|bc `
>方法二：`printf 0x%x n`
>下一步，执行`jstack -l  pid| grep spid -A 100` 打印后面100行分析问题

### 查看/更改系统最大文件打开数
>查看：sysctl fs.file-max 
>更改：sysctl fs.file-max=102400 