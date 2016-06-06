title: java初级：io
date: 2015-12-01 12:02:47
tags:
categories: [初级]
---
### Reader/Writer和InputStream/Output Stream有什么区别?
>一个是字符流，一个是字节流。

###java中有几种类型的流？JDK为每种类型的流提供了一些抽象类以供继承，请说出他们分别是哪些类？
>字节流，字符流。字节流继承于InputStream OutputStream，字符流继承于InputStreamReader OutputStreamWriter。在java.io包中还有许多其他的流，主要是为了提高性能和使用方便。
