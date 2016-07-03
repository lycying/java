title: TIME_WAIT和CLOSE_WAIT解惑
date: 2015-12-07 22:57:39
tags: [net]
categories: [中级]
---

#### 网络状态的获取
Java程序员大多数要与各种网络状态打交道，来处理各种高并发情况下发生的奇怪的问题。其中，TIME_WAIT和CLOSE_WAIT状态是经常碰到问题的状态。但我发现在实际工作中，很少人能定位到此类问题的真正原因。相对于TIME_WAIT，CLOSE_WAIT状态的出现更有可能是程序的BUG引起的。TIME_WAIT的问题需要调整系统参数，但CLOSE_WAIT的出现需要定位代码。

如下代码可以看到系统处于各种网络状态的状态数量。
```bash
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'    
```
可能会得到以下结果：
```bash
TIME_WAIT 18140
CLOSE_WAIT 2882
FIN_WAIT1 1
ESTABLISHED 19829
SYN_RECV 2
LAST_ACK 1
```
关于系统状态的含义，请参见[TCP/IP状态含义](/2015/12/07/tcp-ip-state/)，我们这里只关注TIME_WAIT和CLOSE_WAIT，**TIME_WAIT 表示主动关闭，CLOSE_WAIT 表示被动关**。

Linux的文件句柄是有限的，如果一些状态保持着不消失，将逐渐的耗尽句柄资源，新的连接将直接拒绝，造成异常。
#### 系统保持了大量的CLOSE_WAIT状态
把CLOSE_WAIT放在前面的原因是，出现此状态的原因很可能是程序有问题，根本问题还是程序写的不好，有待提高。
![](/tcp_close.gif)

从上面的图可以看出来，如果一直保持在CLOSE_WAIT状态，那么只有一种情况，就是在**对方关闭连接之后服务器程序自己没有进一步发出ack信号**。换句话说，就是在对方连接关闭之后，程序里没有检测到，或者程序压根就忘记了这个时候需要关闭连接，于是这个资源就一直被程序占着。

拿HttpClient来说，[更详细的分析](http://blog.csdn.net/shootyou/article/details/6615051)：
```
try {
	client = HttpConnectionManager.getHttpClient();
	HttpGet get = new HttpGet();
	get.setURI(new URI(urlPath));
	HttpResponse response = client.execute(get);
	if (response.getStatusLine ().getStatusCode () != 200) {
		return null;
	}
	HttpEntity entity =response.getEntity();

	if( entity != null ){
		in = entity.getContent();
		.....
	}
	return sb.toString ();
} catch (Exception e) { 	return null;
} finally {
	if (isr != null){
		try {
			isr.close ();
		} catch (IOException e) { e.printStackTrace (); }
	}
	if (in != null){
		try {
			in.close();
		} catch (IOException e) { e.printStackTrace (); }
	}
}
```
HttpClient使用我们常用的`InputStream.close()`来确认连接关闭，分析上面的代码，一旦出现`非200`的连接，这个连接将永远僵死在连接池里头，因为inputStream得不到初始化，永远不会调用close()方法了。
正确方法是调用`HttpGet`的`abort()`方法来终止连接。

<div class="tip">一句话，出现了大量CLOSE_WAIT状态，检查下你的代码。</div>
#### 系统保持了大量的TIME_WAIT状态
上面说到TIME_WAIT是主动关闭。什么叫主动关闭呢？就是假如我们建立了一个链接，读取完数据之后，就会发起主动关闭连接。进入TIME_WAIT的状态，然后在保持这个状态2MSL（`max segment lifetime`）。MSL的值在一般的实现中取30s，有些实现采用2分钟。为什么还要等上一段时间呢？虽然双方都同意关闭连接了，而且握手的4个报文也都协调和发送完毕，按理可以直接回到CLOSED状态（就好比从SYN_SEND状态到ESTABLISH状态那样）；但是因为我们必须要假想网络是不可靠的，你无法保证你最后发送的ACK报文会一定被对方收到，因此对方处于LAST_ACK状态下的SOCKET可能会因为超时未收到ACK报文，而重发FIN报文，所以这个TIME_WAIT状态的作用就是用来重发可能丢失的ACK报文，并保证于此。
>TCP下每条连接都有一个属性叫做max segment lifetime，就是说该连接关闭后，要经过2*max segment lifetime的时间，才算是真正的被关闭，才能被重新建立，以防止这条链路上还有东西在传输。

这种情况一般发生在频繁开启和关闭连接的应用中。比如：
- 使用反向代理Nginx
- 有类似爬虫的程序在频繁开启关闭连接
- 某些频繁读写的WEB服务器
- 短时间内接受大量请求或者受到攻击

解决方式：
修改`/etc/sysctl.conf`文件：
在这个文件中，加入下面的几行内容：
```bash
net.ipv4.tcp_syncookies = 1
 #表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
net.ipv4.tcp_tw_reuse = 1
 #表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1
 #表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭；
net.ipv4.tcp_fin_timeout = 5
 #修改系统默认的 TIMEOUT 时间;
```

此外，如果你的连接数本身就很多，我们可以再优化一下`TCP/IP`的可使用端口范围，进一步提升服务器的并发能力。依然是往上面的参数文件中，加入下面这些配置：
```bash
net.ipv4.tcp_keepalive_time = 1200
 #当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为20分钟。
net.ipv4.ip_local_port_range = 10000 65000
 #用于向外连接的端口范围。缺省情况下很小：32768到61000，改为10000到65000。(注意：这里不要将最低值设的太低，否则可能会占用掉正常的端口！)
net.ipv4.tcp_max_syn_backlog = 8192
 #表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。
net.ipv4.tcp_max_tw_buckets = 5000
 #表示系统同时保持TIME_WAIT的最大数量，如果超过这个数字，TIME_WAIT将立刻被清除并打印警告信息。默认为180000，改为5000。
 #对于Apache、Nginx等服务器，上几行的参数可以很好地减少TIME_WAIT套接字数量，但是对于 Squid，效果却不大。
 #此项参数可以控制TIME_WAIT的最大数量，避免Squid服务器被大量的TIME_WAIT拖死。
```
这几个参数，建议只在流量非常大的服务器上开启，会有显著的效果。一般的流量小的服务器上，没有必要去设置这几个参数。

运行 `/sbin/sysctl -p` 使配置生效.

扩展阅读
[在 Windows 上遇到非常多 TIME_WAIT 連線時應如何處理](http://blog.miniasp.com/post/2010/11/17/How-to-deal-with-TIME_WAIT-problem-under-Windows.aspx)
[linux服务器历险之sysctl优化linux网络](http://blog.csdn.net/chinalinuxzend/article/details/1792184)
