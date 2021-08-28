> RabbitMq拒绝消息的三种方式，理解业务需求及三种拒绝方式含义的情况下，谨慎使用

mq中消息的几种状态：

>第一种是等待投递给消费者消费的消息
>
>第二种是已经投递给消费者，等待消费者确认的消息

## basicNack

```java
basicNack(long deliveryTag, boolean multiple, boolean requeue)
```

>deliveryTag:交付标签id,每次递增1，不可做为唯一ID
>
>mulitple:是否拒绝当前所有未确认的消息
>
>requeue:是否重新入队



## basicRecover

```java
basicRecover(boolean requeue)
```

> 拒绝当前消息
>
> true：重新放回队列 ，并且尽可能的投递给其他消费者
>
> false：重新投递给当前消费者

**切记**：当requeue为false时，消息重新重新投递给当前消费者，会有大量循环，系统性能瞬间降低



## baseReject

```java
basicReject(long deliveryTag, boolean requeue)
```

> deliveryTag：较复标签
>
> requeue：是否重新入队



## 区别

basicNack，basicReject，basicRecover三个方法均是拒绝消息，但是应用场景不同

> basicNack 可拒绝批量消息，



```java
public class BizMessageListener implements ChannelAwareMessageListener {
    @Override
    public void onMessage(Message message, Channel channel) throws Exception {
        String string = new String(message.getBody());
        log.info("biz msg:{}", string);
        if (string.endsWith("1")) {
            /**
             * 拒绝本条消息之前的所有未确认的消息，不再重新入队
             * multiple:是否拒绝本条消息之前所有未确认的消息
             * requeue:消息是否重新入队
             */
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), true, false);
        } else if (string.endsWith("2")) {
            //拒绝本条消息，不再重新入队
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
        } else if (string.endsWith("3")) {
            //拒绝消息，重新放回队列
            channel.basicRecover(true);
        } else if (string.endsWith("4")) {
            //消息重新，重新投递给当前消费者
            channel.basicRecover(false);
        } else if (string.endsWith("5")) {
            //拒绝当前消息，重新入队
            channel.basicReject(message.getMessageProperties().getDeliveryTag(), true);
        } else if (string.endsWith("6")) {
            //拒绝当前消息，不重新入队
            channel.basicReject(message.getMessageProperties().getDeliveryTag(), false);
        } else if (string.endsWith("7")) {
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), true);
        } else if (string.endsWith("8")) {
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        }
    }
}
```

