# PHP-FPM
PHP-FPM 即 PHP-FastCGI Process Manager， 它是 FastCGI 的实现，并提供了进程管理的功能。进程包含 master 进程和 worker 进程两种；master 进程只有一个，负责监听端口，接收来自服务器的请求，而 worker 进程则一般有多个（具体数量根据实际需要进行配置），每个进程内部都会嵌入一个 PHP 解释器，是代码真正执行的地方。

## nginx与php-fpm通信方式
在 Linux 上，nginx 与 php-fpm 的通信有 tcp socket 和 unix socket 两种方式。
### tcp socket
tcp socket 的优点是可以跨服务器，当 nginx 和 php-fpm 不在同一台机器上时，只能使用这种方式
### unix socket
>[!NOTE|label:注意]
>在使用 unix socket 方式连接时，由于 socket 文件本质上是一个文件，存在权限控制的问题，所以需要注意 nginx 进程的权限与 php-fpm 的权限问题，不然会提示无权限访问。（在各自的配置文件里设置用户）

Unix socket 又叫 IPC(inter-process communication 进程间通信) socket，用于实现同一主机上的进程间通信，这种方式需要在 nginx配置文件中填写 php-fpm 的 socket 文件位置。
### 区别
由于 Unix socket 不需要经过网络协议栈，不需要打包拆包、计算校验和、维护序号和应答等，只是将应用层数据从一个进程拷贝到另一个进程。所以其效率比 tcp socket 的方式要高，可减少不必要的 tcp 开销。不过，unix socket 高并发时不稳定，连接数爆发时，会产生大量的长时缓存，在没有面向连接协议的支撑下，大数据包可能会直接出错不返回异常。而 tcp 这样的面向连接的协议，可以更好的保证通信的正确性和完整性。

## 在应用中的选择
如果是在同一台服务器上运行的 nginx 和 php-fpm，且并发量不高（不超过1000），选择unix socket，以提高 nginx 和 php-fpm 的通信效率。
如果是面临高并发业务，则考虑选择使用更可靠的 tcp socket，以负载均衡、内核优化等运维手段维持效率。

若并发较高但仍想用 unix socket 时，可通过以下方式提高 unix socket 的稳定性。

1）将sock文件放在 /dev/shm 目录下，此目录下将 sock 文件放在内存里面，内存的读写更快。

2）提高 backlog

backlog 默认位 128，1024 这个值最好换算成自己正常的 QPS，配置如下。


# Nginx&FPM处理流程
```
           开始
            |
            |
 www.example.com/index.php
            |
            |
      nginx 80 端口
            |
            |
  nginx 加载 fastcgi 模块
            |
            |
反向代理到 fpm 监听的 9000 端口
            |
            |
  fpm 处理请求并返回至 nginx 
            |
            |
   nginx 接收并返回客户端
            |
            |
           结束
```