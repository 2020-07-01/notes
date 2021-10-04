> spring 中提供了ConnectionFactory连接和管理RabbitMQ，它是com.rabbitmq.client.Connection类的包装类，一般常用CachingConnectionFactory实现类
>
> 无论生产者和消费者，都需要连接管理器，CachingConnectionFactory主要是用来对连接进行管理
>
> 版本：spring-rabbit 2.2.7



## 属性

### 基础信息

```java
cachingConnectionFactory.setUsername(rabbitMqProperties.getUsername());
cachingConnectionFactory.setPassword(rabbitMqProperties.getPassword());
cachingConnectionFactory.setHost(rabbitMqProperties.getHost());
cachingConnectionFactory.setPort(rabbitMqProperties.getPort());
//集群配置
cachingConnectionFactory.setAddresses(rabbitMqProperties.getAddress());
//设置虚拟主机
cachingConnectionFactory.setVirtualHost(rabbitMqProperties.getVirtualHost());
```



### CacheMode

> CachingConnetionFactory提供两种创建模式

```java
public enum CacheMode {
	//同一个连接，缓存信道；所有客户端之间共享一个TCP连接
   CHANNEL,
	//多个连接和信道；缓存连接和信道
   CONNECTION
}
```

### ConfirmType

> 消息确认模式，新版本的消息自动ack/手动ack模式采用ConfirmType属性配置

```java
private ConfirmType confirmType = ConfirmType.NONE;

public enum ConfirmType {
	//1.启动消息确认模式
    SIMPLE,
	//启动消息确认模式
    CORRELATED,
    //禁用发布确认模式，默认值
    NONE
}
```

### ConnectionCacheSize

> 客户端所能创建的最大连接数量，默认是1
>
> CHANNEL模式下只有一个连接
>
> CONNECTION模式下可以设置多个连接

### ChannelCacheSize

> 一个连接所能创建的最大channel数，默认25
>
> 源码中channelList存储channel，串行操作

```java
private static final int DEFAULT_CHANNEL_CACHE_SIZE = 25;
private int channelCacheSize = DEFAULT_CHANNEL_CACHE_SIZE;
```

### ChannelCheckoutTimeout

> 设置获取信道的超时时间，默认为0，单位毫秒
>
> 大于0时起作用，如果连接上的通道数达到限制时，调用线程会阻塞，指定时间内未获取到channel，则抛出异常
>
> 此配置也作用于获取连接

```java
@Override
public final Connection createConnection() throws AmqpException {
    ......
    synchronized (this.connectionMonitor) {
        if (this.cacheMode == CacheMode.CHANNEL) {
            if (this.connection.target == null) {
                this.connection.target = super.createBareConnection();
                // invoke the listener *after* this.connection is assigned
                if (!this.checkoutPermits.containsKey(this.connection)) {
                    //通过信号量机制控制每个连接下所能缓存的最大信道数量
                    this.checkoutPermits.put(this.connection, new Semaphore(this.channelCacheSize));
                }
                this.connection.closeNotified.set(false);
                getConnectionListener().onCreate(this.connection);
            }
            return this.connection;
        }
        else if (this.cacheMode == CacheMode.CONNECTION) {
            return connectionFromCache();
        }
    }
    return null;
}

private Semaphore obtainPermits(ChannelCachingConnectionProxy connection) {
    Semaphore permits;
    permits = this.checkoutPermits.get(connection);
    if (permits != null) {
        try {
            //获取通道
            if (!permits.tryAcquire(this.channelCheckoutTimeout, TimeUnit.MILLISECONDS)) {				//抛出异常
                throw new AmqpTimeoutException("No available channels");
            }
          ......
    return permits;
}
```

###  ConnectionLimit

> 限制允许的连接总数，默认Integer.MAX_VALUE
>
> 此配置与上面ChannelCheckoutTimeout配置组合起来使用

### PublisherReturns

> 设置消息找不到队列时是否返回



## 实例

```java
@Bean(value = "deadConnectionFactory")
public ConnectionFactory connectionFactory() {
    CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory(rabbitMqProperties.getHost(), rabbitMqProperties.getPort());
    cachingConnectionFactory.setUsername(rabbitMqProperties.getUsername());
    cachingConnectionFactory.setPassword(rabbitMqProperties.getPassword());
    cachingConnectionFactory.setHost(rabbitMqProperties.getHost());
    cachingConnectionFactory.setPort(rabbitMqProperties.getPort());
    //集群配置
    //cachingConnectionFactory.setAddresses(rabbitMqProperties.getAddress());
    //设置虚拟主机
    cachingConnectionFactory.setVirtualHost(rabbitMqProperties.getVirtualHost());
    //设置模式
    cachingConnectionFactory.setCacheMode(CachingConnectionFactory.CacheMode.CHANNEL);
    //设置信道最大缓存数量
    cachingConnectionFactory.setChannelCacheSize(25);
    //设置消息确认模式
  cachingConnectionFactory.setPublisherConfirmType(CachingConnectionFactory.ConfirmType.CORRELATED);
    //设置客户端连接数
    cachingConnectionFactory.setConnectionCacheSize(2);
    //设置获取信道的最大等待时间
    cachingConnectionFactory.setChannelCheckoutTimeout(60 * 1000);
    //设置打开的连接数量
    cachingConnectionFactory.setConnectionLimit(100);
    //设置消息找不到queue是否退回
    cachingConnectionFactory.setPublisherReturns(true);
    return cachingConnectionFactory;
}
```



参考：

https://docs.spring.io/spring-amqp/docs/current/reference/html/