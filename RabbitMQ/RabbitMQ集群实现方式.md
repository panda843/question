
# RabbitMQ集群实现方式

## 主备模式(cluster)
- 不支持跨网段，用于同一个网段内的局域网
- 可以随意的动态增加或者减少
- 节点之间需要运行相同版本的RabbitMQ和Erlang

也称为 Warren (兔子窝) 模式。实现 rabbitMQ 的高可用集群，一般在并发和数据量不高的情况下，这种模式非常的好用且简单。也就是一个主/备方案，主节点提供读写，备用节点不提供读写。如果主节点挂了，就切换到备用节点，原来的备用节点升级为主节点提供读写服务，当原来的主节点恢复运行后，原来的主节点就变成备用节点，和 activeMQ 利用 zookeeper 做主/备一样，也可以一主多备。

## 镜像模式(mirror)
- RabbitMQ的Cluster集群模式一般分为两种，普通模式和镜像模式。

将需要消费的队列变为镜像队列，存在于多个节点，这样就可以实现RabbitMQ的HA高可用性。作用就是消息实体会主动在镜像节点之间实现同步，而不是像普通模式那样，在consumer消费数据时临时读取。缺点就是，集群内部的同步通讯会占用大量的网络带宽。

RabbitMQ主从之间的数据复制是异步的，但是在rabbitmq中不会出现mysql那种丢数据的情况，这是因为rabbitmq的接口也是异步的，主收到一条消息写入本地存储，然后在发起写入从的请求。当所有从写入成功后，主才会给client返回ack说这次写入成功了。所以可以看出，虽然rabbitmq的主从复制是异步的，但是并且不会出现mysql丢数据的场景。只要客户端收到ack，就说明这条消息已经写入主和从了。


## 多活模式(federation)
- 应用于广域网，允许单台服务器上的交换机或队列接收发布到另一台服务器上交换机或队列的消息，可以是单独机器或集群。federation队列类似于单向点对点连接，消息会在联盟队列之间转发任意次，直到被消费者接受。通常使用federation来连接internet上的中间服务器，用作订阅分发消息或工作队列。

federation 插件是一个不需要构建 cluster ，而在 brokers 之间传输消息的高性能插件，federation 插件可以在 brokers 或者 cluster 之间传输消息，连接的双方可以使用不同的 users 和 virtual hosts，双方也可以使用不同版本的 rabbitMQ 和 erlang。federation 插件使用 AMQP 协议通信，可以接受不连续的传输。federation 不是建立在集群上的，而是建立在单个节点上的

## 远程模式(shovel)
- 连接方式与federation的连接方式类似，但它工作在更低层次。可以应用于广域网。

远程模式可以实现双活的一种模式，简称 shovel 模式，所谓的 shovel 就是把消息进行不同数据中心的复制工作，可以跨地域的让两个 MQ 集群互联，远距离通信和复制。
Shovel 就是我们可以把消息进行数据中心的复制工作，我们可以跨地域的让两个 MQ 集群互联。

# 参考
- [RabbitMQ 的4种集群架构](https://www.jianshu.com/p/b7cc32b94d2a)
- [RabbitMQ高可用原理](https://blog.csdn.net/weixin_43737422/article/details/84453899)
- [RabbitMQ主备复制是异步还是同步？](https://www.cnblogs.com/wangzhongqiu/p/7831854.html)
- [RabbitMQ集群搭建-镜像模式](https://www.jianshu.com/p/5b2879fba25b)