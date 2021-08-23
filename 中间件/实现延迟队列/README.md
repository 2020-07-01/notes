





```java
public void declare(@Qualifier RabbitAdmin rabbitAdmin) {

        //声明业务交换机
        Exchange bizExchange = ExchangeBuilder.fanoutExchange("业务交换机").build();

        //死信交换机
        Exchange deadExchange = ExchangeBuilder.directExchange("死信交换机").durable(true).build();

        //声明业务队列A
        Map<String, Object> args = new HashMap<>(16);
        //声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", deadExchange);
        //声明当前队列绑定的死信路由key
        args.put("x-dead-letter-routing-key", "死信路由key");
        Queue bizQueue = QueueBuilder.durable("业务队列A").withArguments(args).build();

        //业务交换机绑定业务队列
        rabbitAdmin.declareQueue(bizQueue);
        rabbitAdmin.declareExchange(bizExchange);
        rabbitAdmin.declareBinding(new Binding(bizQueue.getName(), Binding.DestinationType.QUEUE, bizExchange.getName(), "路由key", Collections.emptyMap()));

        //声明死信队列A
        Queue deadQueue = QueueBuilder.durable("死信队列A").build();
        //死信交换机绑定死信队列
        rabbitAdmin.declareQueue(deadQueue);
        rabbitAdmin.declareExchange(deadExchange);
        rabbitAdmin.declareBinding(new Binding(deadQueue.getName(), Binding.DestinationType.QUEUE, deadExchange.getName(), "路由key", Collections.emptyMap()));
    }
```







参考文章:

https://www.cnblogs.com/mfrank/p/11184929.html#autoid-0-4-0