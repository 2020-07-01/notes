```
ExecutorService
```

* RabbitMQ

  > CachingConnetionFactory 发送消息时以线程池方式处理
  >
  > 线程池默认使用ThreadPoolExecutor

  ```java
  protected ExecutorService getChannelsExecutor() {
      if (getExecutorService() != null) {
          return getExecutorService(); // NOSONAR never null
      }
      if (this.channelsExecutor == null) {
          synchronized (this.connectionMonitor) {
              if (this.channelsExecutor == null) {
                  final String threadPrefix =
                      getBeanName() == null
                      ? DEFAULT_DEFERRED_POOL_PREFIX + threadPoolId.incrementAndGet()
                      : getBeanName();
                  ThreadFactory threadPoolFactory = new CustomizableThreadFactory(threadPrefix); // NOSONAR never null
                  this.channelsExecutor = Executors.newCachedThreadPool(threadPoolFactory);
              }
          }
      }
      return this.channelsExecutor;
  }
  ```

  