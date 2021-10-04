> 对于死信队列自己一直存在疑惑，通过实践，记录此文章，以便之后节省时间不踩坑。

* 死信队列的理解:

  当队列中的消息变成**死信消息**时，自动推送到死信队列中，前提是消息如何变成死信消息，消息如何推送到队列中。

  ---

* 消息变成死信的几种情况：

  * 消费者拒绝并且消息不能重新入队

    >  channel.basicNack(message.getMessageProperties().getDeliveryTag(), true, false);

  * 消息TTL过期

    > 一是消息的过期时间
    >
    > 二是队列的过期时间

  * 队列达到最大长度
  
    > 此处涉及配置队列的**溢出行为**



业务队列需要至少配置两个参数：

> x-dead-letter-exchange：配置死信交换机
>
> x-dead-letter-routing-key：配置死信路由key，通过这个路由到具体的死信队列中



```java
public void declare() {
    //声明业务交换机
    Exchange bizExchange = ExchangeBuilder.fanoutExchange("bizExchange").build();

    //死信交换机
    Exchange deadExchange = ExchangeBuilder.directExchange("deadExchange").durable(true).build();

    //声明业务队列A
    Map<String, Object> args = new HashMap<>(16);
    //声明当前队列绑定的死信交换机
    args.put("x-dead-letter-exchange", deadExchange.getName());
    //声明当前队列绑定的死信路由key 死信交换机路由到指定死信队列
    args.put("x-dead-letter-routing-key", "dead-routing-key");
    Queue bizQueue = QueueBuilder.durable("bizQueue").withArguments(args).build();

    //业务交换机绑定业务队列
    rabbitAdmin.declareQueue(bizQueue);
    rabbitAdmin.declareExchange(bizExchange);
    rabbitAdmin.declareBinding(new Binding(bizQueue.getName(), Binding.DestinationType.QUEUE, bizExchange.getName(), "biz-routing-key", Collections.emptyMap()));

    //声明死信队列A
    Queue deadQueue = QueueBuilder.durable("deadQueue").build();
    //死信交换机绑定死信队列
    rabbitAdmin.declareQueue(deadQueue);
    rabbitAdmin.declareExchange(deadExchange);
    rabbitAdmin.declareBinding(new Binding(deadQueue.getName(), Binding.DestinationType.QUEUE, deadExchange.getName(), "dead-routing-key", Collections.emptyMap()));
}
```



- 消费端拒绝接受消息并且消息不能再次入队

  ```java
  public class BizMessageListener implements ChannelAwareMessageListener {
      @Override
      public void onMessage(Message message, Channel channel) throws Exception {
          String string = new String(message.getBody());
          log.info("biz msg:{}", string);
          if (string.endsWith("nack")) {
              //拒绝重新入队
              channel.basicNack(message.getMessageProperties().getDeliveryTag(), true, false);
          } else {
              channel.basicAck(message.getMessageProperties().getDeliveryTag(), true);
          }
      }
  }
  ```

  

参考文章:

https://www.cnblogs.com/mfrank/p/11184929.html#autoid-0-4-0