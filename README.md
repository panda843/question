## PHP
- [PHP-Interview-QA](https://github.com/colinlet/PHP-Interview-QA)
- [php-interview-2018](https://github.com/sushengbuhuo/php-interview-2018)
- [PHPerInterviewGuide](https://github.com/todayqq/PHPerInterviewGuide)
- [php-engineer-interview-questions](https://github.com/hookover/php-engineer-interview-questions)
- [interview](https://github.com/wanglelecc/interview)
- [php-be-intervie](https://github.com/rucblake/php-be-interview)
- [PHP面试题及答案](https://github.com/lengyue1024/BAT_interviews/blob/master/PHP%E9%9D%A2%E8%AF%95%E9%A2%98%E5%8F%8A%E7%AD%94%E6%A1%88.md)
- [mysql优化面试题](mysql优化面试题.pdf)
- [PHP8新特性之JIT简介](https://www.laruence.com/2020/06/27/5963.html)
- [PHP8新特性盘点](https://blog.thinkphp.cn/2052124)
- [PHP7新特性](https://www.runoob.com/w3cnote/php7-new-features.html)
- [CGI、FastCGI和PHP-FPM关系图解](https://www.awaimai.com/371.html)
- [PHP7为什么比5快](https://blog.csdn.net/changfangyuansh/article/details/102692923)


## 消息队列
- [消息队列高频问题](http://www.mianshigee.com/tag/messagequeen/s1/)
- [2021最新 RabbitMQ面试题精选](https://blog.csdn.net/lupengfei1009/article/details/114525479)

## 缓存
- [redis中跳表原理&实现](https://blog.csdn.net/mffandxx/article/details/112284405)
- [码上一波 Redis 面试题，面试跳槽不要慌!](https://blog.csdn.net/weixin_50333534/article/details/112447835?spm=1001.2014.3001.5501)
- [2021一波最新 Redis 面试题](https://blog.csdn.net/weixin_50333534/article/details/112447879?spm=1001.2014.3001.5501)
- [2021年最新版 68道Redis面试题](https://www.php.cn/faq/457390.html)
- [史上最全的50个Redis面试题及答案](https://www.php.cn/redis/436652.html)
- [2021年Redis面试题（持续更新）](https://blog.csdn.net/qq1515312832/article/details/113880849)

## Linux

### 统计nginx日志里访问次数最多的前十个IP
```bash
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr -k1 | head -n 10
```

## 常见算法
- [数据结构与算法系列目录](https://www.cnblogs.com/skywang12345/p/3603935.html)

### 树常见面试题
- [红黑树常见面试问题整理](https://blog.csdn.net/fantacy10000/article/details/100855055)
- [红黑树之原理和算法详细介绍](https://www.cnblogs.com/skywang12345/p/3245399.html)
- [B树，B+树，红黑树 数据库常见面试题](https://blog.csdn.net/zhangshk_/article/details/83013482)
- [腾讯、阿里面试题 了解B+树吗？](https://blog.csdn.net/zycxnanwang/article/details/105384618)

### 二分查找

```php
#二分查找
function binarySearch(Array $arr, $target) {
    $low = 0;
    $high = count($arr) - 1;
        
    while($low <= $high) {
        $mid = floor(($low + $high) / 2);
        #找到元素
        if($arr[$mid] == $target) return $mid;
        #中元素比目标大,查找左部
        if($arr[$mid] > $target) $high = $mid - 1;
        #重元素比目标小,查找右部
        if($arr[$mid] < $target) $low = $mid + 1;
    }
    #查找失败
    return false;
}
```

### 冒泡排序

```php
$demo_array = array(23,15,43,25,54,2,6,82,11,5,21,32,65);
// 第一层for循环可以理解为从数组中键为0开始循环到最后一个
for ($i=0;$i<count($demo_array);$i++) {
    // 第二层将从键为$i的地方循环到数组最后
    for ($j=$i+1;$j<count($demo_array);$j++) {
        // 比较数组中相邻两个值的大小
        if ($demo_array[$i] > $demo_array[$j]) {
            $tmp            = $demo_array[$i]; // 这里的tmp是临时变量
            $demo_array[$i] = $demo_array[$j]; // 第一次更换位置
            $demo_array[$j] = $tmp;            // 完成位置互换
        }
    }
}
```

### 快速排序

```php
function quick_sort($arr){
    //先判断是否需要继续进行
    $length = count($arr);
    if($length <= 1) {
        return $arr;
    }
    //选择第一个元素作为基准
    $base_num = $arr[0];
    //遍历除了标尺外的所有元素，按照大小关系放入两个数组内
    //初始化两个数组
    $left_array = array();  //小于基准的
    $right_array = array();  //大于基准的
    for($i=1; $i<$length; $i++) {
        if($base_num > $arr[$i]) {
            //放入左边数组
            $left_array[] = $arr[$i];
        } else {
            //放入右边
            $right_array[] = $arr[$i];
        }
    }
    //再分别对左边和右边的数组进行相同的排序处理方式递归调用这个函数
    $left_array = quick_sort($left_array);
    $right_array = quick_sort($right_array);
    //合并
    return array_merge($left_array, array($base_num), $right_array);;
}
```

## 常见问题

### Mysql回表原理

```
MySQL innodb的主键索引是簇集索引，也就是索引的叶子节点存的是整个单条记录的所有字段值，不是主键索引的就是非簇集索引，非簇集索引的叶子节点存的是主键字段的值。回表是什么意思？就是你执行一条sql语句，需要从两个b+索引中去取数据。举个例子：

表tbl有a,b,c三个字段，其中a是主键，b上建了索引，然后编写sql语句

SELECT * FROM tbl WHERE a=1

这样不会产生回表，因为所有的数据在a的索引树中均能找到

SELECT * FROM tbl WHERE b=1

这样就会产生回表，因为where条件是b字段，那么会去b的索引树里查找数据，但b的索引里面只有a,b两个字段的值，没有c，那么这个查询为了取到c字段，就要取出主键a的值，然后去a的索引树去找c字段的数据。查了两个索引树，这就叫回表。

索引覆盖就是查这个索引能查到你所需要的所有数据，不需要去另外的数据结构去查。其实就是不用回表。

怎么避免？不是必须的字段就不要出现在SELECT里面。或者b,c建联合索引。但具体情况要具体分析，索引字段多了，存储和插入数据时的消耗会更大。这是个平衡问题。
```

### 常见协议层

```
OSI分层(7层)
    物理层 -> 数据链路层 -> 网络层 -> 运输层 -> 会话层 -> 表示层 -> 应用层
五层协议(是OSI和TCP/IP的综合)
    物理层 -> 数据链路层 -> 网络层 -> 运输层 -> 应用层
TCP/IP(4层)
    网络接口层 -> 网际层 -> 运输层 -> 应用层


1、物理层：
主要定义物理设备标准，如网线的接口类型、光纤的接口类型、各种传输介质的传输速率等。它的主要作用是传输比特流（就是由1、0转化为电流强弱来进行传输,到达目的地后在转化为1、0，也就是我们常说的数模转换与模数转换）。这一层的数据叫做比特。 　　

2、数据链路层：
定义了如何让格式化数据以进行传输，以及如何让控制对物理介质的访问。这一层通常还提供错误检测和纠正，以确保数据的可靠传输。 　　

3、网络层：
在位于不同地理位置的网络中的两个主机系统之间提供连接和路径选择。Internet的发展使得从世界各站点访问信息的用户数大大增加，而网络层正是管理这种连接的层。 　　

4、运输层：
定义了一些传输数据的协议和端口号（WWW端口80等），如： 
TCP（transmission control protocol –传输控制协议，传输效率低，可靠性强，用于传输可靠性要求高，数据量大的数据） 
UDP（user datagram protocol–用户数据报协议，与TCP特性恰恰相反，用于传输可靠性要求不高，数据量小的数据，如QQ聊天数据就是通过这种方式传输的）。 主要是将从下层接收的数据进行分段和传输，到达目的地址后再进行重组。常常把这一层数据叫做段。 　　

5、会话层：
通过运输层（端口号：传输端口与接收端口）建立数据传输的通路。主要在你的系统之间发起会话或者接受会话请求（设备之间需要互相认识可以是IP也可以是MAC或者是主机名） 　　

6、表示层：
可确保一个系统的应用层所发送的信息可以被另一个系统的应用层读取。例如，PC程序与另一台计算机进行通信，其中一台计算机使用扩展二一十进制交换码（EBCDIC），而另一台则使用美国信息交换标准码（ASCII）来表示相同的字符。如有必要，表示层会通过使用一种通格式来实现多种数据格式之间的转换。 　　

7、应用层：
是最靠近用户的OSI层。这一层为用户的应用程序（例如电子邮件、文件传输和终端仿真）提供网络服务。    
```

### TCP三次握手
```
1. 客户端发送一个TCP的SYN=1的包(SYN=1, seq=客户端序列号) 客户端进入SYN_SEND状态 
2. 服务器发回ACK确认包(SYN=1, ACK=1, seq=服务端序列号, ACKnum(确认序号)=客户的ISN加1) 服务器端进入SYN_RCVD状态 
3. 客户端再次发送ACK确认包(ACK=1，seq=客服端的ISN+1 ACKnum(确认序号)=服务端的ISN加1) 客户端进入ESTABLISHED(已确认)状态,服务端收到到后进入ESTABLISHED(已确认)状态 
```

### TCP四次挥手

```
1. 客服端发送FIN=1标志位置为1的包(FIN=1，seq=x) 客户端进入FIN_WAIT_1状态
2. 服务器端发送一个ACK确认包(ACK=1，ACKnum=x+1) 服务器端进入CLOSE_WAIT状态 (客户端接收到这个确认包之后，进入FIN_WAIT_2状态)
3. 服务器端发送一个结束连接请求(FIN=1，seq=y) 服务器端进入LAST_ACK状态(等待来自客户端的最后一个ACK)
4. 客户端接收到后发送一个ACK确认包，并进入TIME_WAIT状态(可能出现的要求重传的ACK包) 等待了某个固定时间 进入CLOSED状态
```

### Cookie&Session区别
```
存储位置不同 (cookie放在客服端,session在服务端)
存储容量不同 (cookie大小受浏览器限制,对于session来说并没有上限)
隐私策略不同 (cookie对客户端是可见的,session存储在服务器上)
```

###  Http报文格式

```
请求
    请求行：请求方法 + URL + 协议/版本号
    请求头部：浏览器告知给服务端的属性信息
    空行
    请求主体：请求相关的数据信息（参数等）
响应
    响应行：协议/版本号 + 状态码 + 描述信息
    响应头部：服务端告知浏览器的属性信息
    空行
    响应主体：响应数据(html文件等)    
```

### 一个完整的HTTP&HTTPS请求

```
http:
    域名解析 -> 发起TCP的3次握手 -> 建立TCP连接后发起http请求 -> 服务器响应http请求 -> TCP连接关闭 -> 浏览器得到html代码 -> 渲染
https(HTTPS跟HTTP一样，只不过增加了SSL):
    验证服务器端 -> 允许客户端和服务器端选择加密算法和密码，确保双方都支持 -> 验证客户端(可选) -> 使用公钥加密技术来生成共享加密数据 -> 创建一个加密的SSL连接
-> 基于该 SSL 连接传递 HTTP 请求
```

### 对称加密和非对称加密的区别
```
对称加密:  是最快速、最简单的一种加密方式, 加密与解密用的是同样的密钥
非对称加密:  
1. A要向B发送信息，A和B都要产生一对用于加密和解密的公钥和私钥。
2. A的私钥保密，A的公钥告诉B；B的私钥保密，B的公钥告诉A。
3. A要给B发送信息时，A用B的公钥加密信息，因为A知道B的公钥。
4. A将这个消息发给B（已经用B的公钥加密消息）。
5. B收到这个消息后，B用自己的私钥解密A的消息，其他所有收到这个报文的人都无法解密，因为只有B才有B的私钥。
6. 反过来，B向A发送消息也是一样。

```

### 常见状态码

>参考
- [Http状态码](其他/Http状态码.md)

```
400：bad requst - 通常是请求语法问题
403：forbidden - 认证授权未通过
404：not found - 资源不存在
500：Internal server error - 不可预期的错误
502：gateway bad - 收到后端无效响应
503：service unavaible - 由于临时的服务器维护或者过载
504：gateway timeout - 服务器尝试执行请求时未能及时收到响应
499：认为是不安全的连接，主动拒绝了客户端的连接
```

### GET和POST方法区别
```
数据存储位置: Get方法放在URL中；Post方法放到body中
数据大小限制: 浏览器对URL长度有限制
安全性: 用户名和密码等参数出现在URL上存在安全隐患
数据类型: GET只允许ASCII字符,POST没有限制
```

### HTTP1.1和HTTP2.0的区别
```
多路复用：连接共享机制让一个连接让可以有多个request请求，服务端通过request请求id区分后让不同的服务端进行响应，让原本的串行的请求-响应模式变为并行，提高传输处理效率
二进制格式：让原本基于文本格式的解析方法变为二进制格式的解析方法，以此来提高解析报文的健壮性
header压缩：通过缓存头部字段来减少传输大小和避免重复发送header字段问题
服务端推送：把客户端所需要的资源伴随着index.html一起发送到客户端，省去了客户端重复请求的步骤，以此提高性能
长连接：同一个TCP连接可以承载多个http请求和响应；而多路复用指的是同一个http连接可以共享
```

### 秒杀&超卖问题

>参考
- [https://blog.csdn.net/zhoudaxia/article/details/38067003](https://blog.csdn.net/zhoudaxia/article/details/38067003)
- [秒杀系统的设计与实现](PHP/秒杀系统的设计与实现.md)


```
分两个步骤解决:
1. 前端
    扩容(加机器，这是最简单的方法，通过增加前端机器来抗峰值) 
    静态化(将页面上的所有可以静态的元素全部静态化，并尽量减少动态元素。通过CDN来抗峰值。)
    限流(可以通过限制 IP,按钮,用户 ID 等来限制单人短时间内发起的请求数量)
    拒绝请求(随机拒绝部分请求来保护整体的可用性,可以使用 nginx 进行限流)
2. 后端
    业务分离(将秒杀业务系统和其他业务分离，单独放在高配服务器上，可以集中资源对访问请求抗压)
    使用缓存(热点数据都从缓存获得，尽可能减小数据库的访问压力)
    库存前置(放入内存中异步同步到数据库,可以使用队列或Redis事务进行减库存操作)
    使用队列(商品放入队列中,来一个人取一个过程中有操作失败,可重新返回队列)
    分流(部署多台机器,通过负载均衡共同处理客户端请求，分散压力。)
    异步(如发短信、更新数据库等，从而缓解服务器峰值压力。)    
```
