title: java中级：网络开发
date: 2015-12-01 12:04:56
tags:
categories: [中级]
---
### SO_REUSEADDR是为了解决什么问题?
>防止服务器在发生意外时，端口未被释放，可以重新使用。

### Socket编程中，你都遇到了什么网络异常。
>大体这些，考察是否真正使用过：
java.net.BindException:Address already in use: JVM_Bind
java.net.ConnectException: Connection refused: connect
java.net.SocketException: Socket is closed
java.net.SocketException: （Connection reset)
java.net.SocketException: Broken pipe

### SO_LINGER选项是什么意思？
>这个Socket选项可以影响close方法的行为。在默认情况下，当调用close方法后，将立即返回；如果这时仍然有未被送出的数据包，那么这些数据包将被丢弃。如果将linger参数设为一个正整数n时（n的值最大是65，535），在调用close方法后，将最多被阻塞n秒。在这n秒内，系统将尽量将未送出的数据包发送出去；如果超过了n秒，如果还有未发送的数据包，这些数据包将全部被丢弃；而close方法会立即返回。如果将linger设为0，和关闭SO_LINGER选项的作用是一样的。

### SO_TIMEOUT选项是什么意思?
>这个Socket选项可以通过这个选项来设置读取数据超时。当输入流的read方法被阻塞时，如果设置timeout（timeout的单位是毫秒），那么系统在等待了timeout毫秒后会抛出一个InterruptedIOException例外。在抛出例外后，输入流并未关闭，你可以继续通过read方法读取数据。

### SO_SNDBUF，SO_RCVBUF是干什么用的？
>在默认情况下，输出流的发送缓冲区是8096个字节（8K）。这个值是Java所建议的输出缓冲区的大小。如果这个默认值不能满足要求，可以用setSendBufferSize方法来重新设置缓冲区的大小。但最好不要将输出缓冲区设得太小，否则会导致传输数据过于频繁，从而降低网络传输的效率。

### SO_KEEPALIVE是干什么用的？
>如果将这个Socket选项打开，客户端Socket每隔段的时间（大约两个小时）就会利用空闲的连接向服务器发送一个数据包。这个数据包并没有其它的作用，只是为了检测一下服务器是否仍处于活动状态。如果服务器未响应这个数据包，在大约11分钟后，客户端Socket再发送一个数据包，如果在12分钟内，服务器还没响应，那么客户端Socket将关闭。如果将Socket选项关闭，客户端Socket在服务器无效的情况下可能会长时间不会关闭。

### SO_OOBINLINE选项是干什么用的？
>如果这个Socket选项打开，可以通过Socket类的sendUrgentData方法向服务器发送一个单字节的数据。这个单字节数据并不经过输出缓冲区，而是立即发出。虽然在客户端并不是使用OutputStream向服务器发送数据，但在服务端程序中这个单字节的数据是和其它的普通数据混在一起的。因此，在服务端程序中并不知道由客户端发过来的数据是由OutputStream还是由sendUrgentData发过来的。