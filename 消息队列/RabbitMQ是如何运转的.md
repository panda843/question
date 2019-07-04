# MQ的作用
1. 解耦：可以很好地屏蔽应用程序及平台之间的特性，充当中间者，松散耦合应用程序及平台，它们彼此不需要了解远程过程调用RPC与网络协议的细节；
2. 异步通信：能提供C/S之间同步与异步连接，在任何时刻都可以将消息进行传送或者存储转发；
3. 可恢复性：当消息接收方宕机或网络不通的情况下，消息转储于MQ中，直到网络恢复或接收方恢复再进行转发；
4. 扩展性：提高消息入队列和处理效率是容易的，只需要另外增加处理过程即可，不需要改变代码，也不需要调节参数。
5. 顺序性：由于大部分MQ支持队列模式，自然也就能保证一定的数据处理顺序（事实上是不准确的，很有局限性。消息的顺序性取决于很多因素）
6. 缓冲：MQ通过一个缓冲层来帮助任务最高效率执行，写入MQ的处理会尽可能快速。

# RabbitMQ的消息处理过程
![交换机模型](https://www.cloudamqp.com/img/blog/exchanges-topic-fanout-direct.png)

# RabbitMQ的四种交换机
## 直连交换机：Direct exchange
>[!TIP|label:适用场景]
>有优先级的任务，根据任务的优先级把消息发送到对应的队列，这样可以指派更多的资源去处理高优先级的队列。

直连交换机是一种带路由功能的交换机，一个队列会和一个交换机绑定，除此之外再绑定一个routing_key，当消息被发送的时候，需要指定一个binding_key，这个消息被送达交换机的时候，就会被这个交换机送到指定的队列里面去。同样的一个binding_key也是支持应用到多个队列中的。这样当一个交换机绑定多个队列，就会被送到对应的队列去处理。
![直连交换机](http://www.yanzuoguang.com/upload/2019/03/qv737hq5rsgnhp75obrd7hshqj.png)

## 扇形交换机：Fanout exchange
广播消息。扇形交换机会把能接收到的消息全部发送给绑定在自己身上的队列。因为广播不需要“思考”，所以扇形交换机处理消息的速度也是所有的交换机类型里面最快的。
![扇形交换机](http://www.yanzuoguang.com/upload/2019/03/6geblh5ah2jk2rrtof2rfpksm7.png)

## 主题交换机：Topic exchange
RabbitMQ提供了一种主题交换机，发送到主题交换机上的消息需要携带指定规则的routing_key，主题交换机会根据这个规则将数据发送到对应的(多个)队列上。
主题交换机的routing_key需要有一定的规则，交换机和队列的binding_key需要采用*.#.*.....的格式，每个部分用.分开
- *表示一个单词
- #表示任意数量（零个或多个）单词。
![主题交换机](http://www.yanzuoguang.com/upload/2019/03/us3brfibvaiugr6n5iolpv3dbr.png)
## 首部交换机：Headers exchange
- 首部交换机是忽略routing_key的一种路由方式。路由器和交换机路由的规则是通过Headers信息来交换的，这个有点像HTTP的Headers。将一个交换机声明成首部交换机，绑定一个队列的时候，定义一个Hash的数据结构，消息发送的时候，会携带一组hash数据结构的信息，当Hash的内容匹配上的时候，消息就会被写入队列。
- 绑定交换机和队列的时候，Hash结构中要求携带一个键“x-match”，这个键的Value可以是any或者all，这代表消息携带的Hash是需要全部匹配(all)，还是仅匹配一个键(any)就可以了。相比直连交换机，首部交换机的优势是匹配的规则不被限定为字符串(string)。

# 生产者流程　　
1. 生产者连接到RabbitMQ Broker，建立Connection，开启信道Channel（Connection与Channel概念下面会介绍）
2. 生产者声明一个交换器，设置相关属性。
3. 生产者声明一个队列并设置相关属性
4. 生产者通过路由键将交换器和队列绑定起来
5. 生产者发送消息到RabbitMQ Broker，包括路由键、交换器信息等
6. 相应的交换器根据路由键查找匹配的队列
7. 如果找到则消息存入相应队列中
8. 如果没找到则根据配置的属性丢弃或者回退给生产者
9. 关闭信道
10. 关闭连接

# 消费者流程
1. 消费者连接到RabbitMQ Broker，建立Connection，开启Channel
2. 消费者向RabbitMQ Broker请求消费相应队列中消息，可能会设置相应的回调函数。
3. 等待RabbitMQ Broker回应并投递相应队列中的消息，消费者接收消息。
4. 消费者确认ack接收到的消息。
5. RabbitMQ从队列中删除相应已经被确认的消息。
6. 关闭信道。
7. 关闭连接

# Connection与Channel概念
- Connection：实际就是一条TCP连接，TCP一旦建立起来，客户端紧接着可以创建AMQP信道。
- Channel：每个Channel都有唯一的ID，都是建立在Connection上的虚拟连接，RabbitMQ处理每条AMQP指令都是通过信道完成的

>[!TIP|label:单TCP复用连接与多信道的优势]
> 1. 为什么TCP连接只有一条，而每个生产者都会创建一条唯一的信道呢？想象下，实际情况，会有很多的生产者生产消息，多个消费者消费消息，那么就不得不创建多个线程，建立多个TCP连接。多个TCP连接的建立必然会对操作系统性能消耗较高，也不方便管理。从而选择一种类似于NIO（非阻塞I/O, Non-blocking I/O）技术是很有必要的，多信道的在TCP基础上的建立就是这么实现的。
> 2. 每个线程都有自己的一个信道，复用了Connection的TCP连接，信道之间相互独立，相互保持神秘，节约TCP连接资源，当然本身信道的流量很大的话，也可以创建多个适当的Connection的TCP连接，需要根据具体业务情况制定。

![Connection&Channel原理图](http://image.morecoder.com/article/201810/20181001154334353562.png)


# 参考
- [RabbitMQ的四种交换机](https://www.jianshu.com/p/469f4608ce5d)
- [RabbitMQ是如何运转的？](https://www.cnblogs.com/jian0110/p/10389986.html)
- [认识RabbitMQ交换机模型](https://www.cnblogs.com/jian0110/p/10389780.html)
