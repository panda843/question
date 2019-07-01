# 生产者丢数据
确认机制相对于事务机制，最大的好处就是可以异步处理提高吞吐量，不需要额外等待消耗资源。但是两者时候不能同时共存的。

## 事务机制
>[!TIP|label:说明]
>与事务机制相关的有三种方法，分别是channel.txSelect设置当前信道为事务模式、channel.txCommit提交事务和channel.txRollback事务回滚。如果事务提交成功，则消息一定是到达了RabbitMQ中，如果事务提交之前由于发送异常或者其他原因，捕获后可以进行channel.txRollback回滚。

```java
// 将信道设置为事务模式，开启事务
channel.txSelect(); 
// 发送持久化消息
channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, MessageProperties.PERSISTENT_TEXT_PLAIN, "transaction messages".getBytes()); 
// 事务提交
channel.txCommit();
// 事物回滚
channel.txRollback();
```

## 消息确认机制
>[!TIP|label:说明]
> - 生产者将信道设置为confirm确认模式，确认之后所有在信道上的消息将会被指派一个唯一的从1开始的ID，一旦消息被正确匹配到所有队列后，RabbitMQ就会发送一个确认Basic.Ack给生产者（包含消息的唯一ID），生产者便知晓消息是否正确到达目的地了。
> - 消息如果是持久化的，那么确认消息会在消息写入磁盘之后发出。RabbitMQ中的deliveryTag包含了确认消息序号，还可以设置multiple参数，表示到这个序号之前的所有消息都已经得到处理。确认机制相对事务机制来说，相比较代码来说比较复杂，但会经常使用，主要有单条确认、批量确认、异步批量确认三种方式。

### 单条确认
```java
// 设置 channel 消息推送为 单条确认模式 
channel.confirmSelect();
// 推送消息
channel.basicPublish("exchange", "routingkey", null, "publisher confirm test".getBytes());
// 消息推送失败处理
if (!channel.waitForConfirms()) {
    //publisher confirm failed handle
}
```

### 批量确认
>[!TIP|label:问题]
> 批量确认comfirm需要解决出现返回的Basic.Nack或者超时情况的消息全部重发怎么解决

```java
# 增加一个缓存，将发送成功并且确认Ack之后的消息去除，剩下Nack或者超时的消息
// take ArrayList or BlockingQueue as a cache
List<Object> cache = new ArrayList<>();
// set channel publisher confirm mode
channel.confirmSelect();
for (int i=0; i < 20; i++) {
    // publish message
    String message = "publisher message["+ i +"]";
    cache.add(message);
    channel.basicPublish("exchange", "routingkey", null, message.getBytes());
    if (channel.waitForConfirms()) {
        // remove message publisher confirm
        cache.remove(i);
    }
    // TODO handle Nack message：republish
}
```

### 异步批量确认
>[!NOTE|lable:说明]
>异步确认方式通过在客户端addConfirmListener增加ConfirmListener回调接口，包括handleAck与handleNack处理方法

```java
// take  as a cache
SortedSet cache = new TreeSet();
// set channel publisher confirm mode
channel.confirmSelect();
for (int i = 0; i < 20; i++) {
    // publish message
    long nextSeqNo = channel.getNextPublishSeqNo();
    String message = "publisher message[" + i + "]";
    cache.add(message);
    channel.basicPublish("exchange", "routingkey", MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
    cache.add(nextSeqNo);
}
// add confirmCalback: handleAck, handleNack
channel.addConfirmListener(new ConfirmListener() {
    @Override
    public void handleAck(long deliveryTag, boolean multiple) {
        if (multiple) {
            // batch remove ack message
            cache.headSet(deliveryTag - 1).clear();
        } else {
            // remove ack message
            cache.remove(deliveryTag);
        }
    }
    @Override
    public void handleNack(long deliveryTag, boolean multiple) {
            // TODO handle Nack message：republish
    }
});
```

## 参考
- [四种途径提高RabbitMQ传输消息数据的可靠性（一）](https://www.cnblogs.com/jian0110/p/10419013.html)
- [四种途径提高RabbitMQ传输数据的可靠性（二）](https://www.cnblogs.com/jian0110/p/10424927.html)
- [RabbitMQ如何解决各种情况下丢数据的问题](https://www.cnblogs.com/tinyj/p/9977205.html)