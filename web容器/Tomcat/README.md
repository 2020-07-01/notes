> 之前对Tomcat源码做过一些研究，很多已忘记，最近在研究线程模型，特此将自己对Tomcat的学习做一个总结

## 版本

最新版本tomcat10

官方文档：https://tomcat.apache.org/tomcat-8.5-doc/config/http.html



## 线程模型

* BIO

  > tomcat8.5之后舍弃了该模型（JioEndPoint)

* NIO

  > tomcat8.5之后的默认模型（NioEndPoint）

* NIO2

  > 异步非阻塞IO模型

* apr

  > 待研究

## 调优参数

[Apache Tomcat 8 Configuration Reference](https://tomcat.apache.org/tomcat-8.5-doc/config/http.html)



* acceptCount

  > 与操作系统对socket的处理有关



## 部分源码

### LimitLatch

> 控制tomcat某个时刻最大连接数，当请求连接数达到最大时，阻塞
>
> 通过LimitLatch(AQS)实现
>
> 配置参数：maxConnections

```java
//org.apache.tomcat.util.net.AbstractEndpoint#setMaxConnections
private int maxConnections = 8*1024;
//初始化
public void setMaxConnections(int maxCon) {
    this.maxConnections = maxCon;
    LimitLatch latch = this.connectionLimitLatch;
    if (latch != null) {
        // Update the latch that enforces this
        if (maxCon == -1) {
            releaseConnectionLatch();
        } else {
            latch.setLimit(maxCon);
        }
    } else if (maxCon > 0) {
        initializeConnectionLatch();
    }
}

//org.apache.tomcat.util.net.Acceptor#run
//当请求数达到最大时，阻塞
endpoint.countUpOrAwaitConnection();
//org.apache.tomcat.util.threads.LimitLatch#countUpOrAwait
```



### Acceptor

> 接受请求

```java
//org.apache.tomcat.util.net.Acceptor#run
//设置socket
endpoint.setSocketOptions
```



### NioEndPoint

> 请求处理核心类

```java
//org.apache.tomcat.util.net.NioEndpoint#setSocketOptions
//设置socket未非阻塞模式
socket.configureBlocking(false);
```



### Executor

> Tomcat内置定制版线程池StandardThreadExecutor
>
> 阻塞队列为无限队列
>
> 处理任务的默认最大线程为200
>
> 配置参数：maxThreads

```java
//org.apache.tomcat.util.net.NioEndpoint#startInternal
 if (getExecutor() == null) {
     createExecutor();
 }
//org.apache.tomcat.util.net.AbstractEndpoint#createExecutor
//创建线程池
public void createExecutor() {
    internalExecutor = true;
    TaskQueue taskqueue = new TaskQueue();
    TaskThreadFactory tf = new TaskThreadFactory(getName() + "-exec-", daemon, getThreadPriority());
    executor = new ThreadPoolExecutor(getMinSpareThreads(), getMaxThreads(), 60, TimeUnit.SECONDS,taskqueue, tf);
    taskqueue.setParent( (ThreadPoolExecutor) executor);
}

//org.apache.tomcat.util.net.AbstractEndpoint#setMaxThreads
private int maxThreads = 200;
public void setMaxThreads(int maxThreads) {
    this.maxThreads = maxThreads;
    Executor executor = this.executor;
    if (internalExecutor && executor instanceof java.util.concurrent.ThreadPoolExecutor) {
        // The internal executor should always be an instance of
        // j.u.c.ThreadPoolExecutor but it may be null if the endpoint is
        // not running.
        // This check also avoids various threading issues.
        ((java.util.concurrent.ThreadPoolExecutor) executor).setMaximumPoolSize(maxThreads);
    }
}
```



### acceptCount

> 服务器可以同时响应的客户端请求数

```java
private int acceptCount = 100;
public void setAcceptCount(int acceptCount) { if (acceptCount > 0) this.acceptCount = acceptCount; }
public int getAcceptCount() { return acceptCount; }

//org.apache.tomcat.util.net.NioEndpoint#initServerSocket
protected void initServerSocket() throws Exception {
    if (!getUseInheritedChannel()) {
        serverSock = ServerSocketChannel.open();
        socketProperties.setProperties(serverSock.socket());
        InetSocketAddress addr = new InetSocketAddress(getAddress(), getPortWithOffset());
        serverSock.socket().bind(addr,getAcceptCount());
    } else {
        // Retrieve the channel provided by the OS
        Channel ic = System.inheritedChannel();
        if (ic instanceof ServerSocketChannel) {
            serverSock = (ServerSocketChannel) ic;
        }
        if (serverSock == null) {
            throw new IllegalArgumentException(sm.getString("endpoint.init.bind.inherited"));
        }
    }
    //
    serverSock.configureBlocking(true); //mimic APR behavior
}
```



### ServerSocketChannel

> socket的超时时间默认为6000

```java
//org.apache.tomcat.util.net.NioEndpoint#initServerSocket
protected void initServerSocket() throws Exception {
    if (!getUseInheritedChannel()) {
        serverSock = ServerSocketChannel.open();
        //设置属性 
        socketProperties.setProperties(serverSock.socket());
        InetSocketAddress addr = new InetSocketAddress(getAddress(), getPortWithOffset());
        serverSock.socket().bind(addr,getAcceptCount());
    } else {
        // Retrieve the channel provided by the OS
        Channel ic = System.inheritedChannel();
        if (ic instanceof ServerSocketChannel) {
            serverSock = (ServerSocketChannel) ic;
        }
        if (serverSock == null) {
            throw new IllegalArgumentException(sm.getString("endpoint.init.bind.inherited"));
        }
    }
    serverSock.configureBlocking(true); //mimic APR behavior
}


public final class Constants {
    //socket 默认超时时间
    public static final int DEFAULT_CONNECTION_TIMEOUT = 60000;
}
```



### processSocket

> 处理请求
>
> 此过程是放在上面所创建的线程池中执行

```java
public boolean processSocket(SocketWrapperBase<S> socketWrapper,
                             SocketEvent event, boolean dispatch) {
    try {
        if (socketWrapper == null) {
            return false;
        }
        SocketProcessorBase<S> sc = null;
        if (processorCache != null) {
            sc = processorCache.pop();
        }
        if (sc == null) {
            sc = createSocketProcessor(socketWrapper, event);
        } else {
            sc.reset(socketWrapper, event);
        }
        Executor executor = getExecutor();
        if (dispatch && executor != null) {
            executor.execute(sc);
        } else {
            sc.run();
        }
    } catch (RejectedExecutionException ree) {
        getLog().warn(sm.getString("endpoint.executor.fail", socketWrapper) , ree);
        return false;
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        // This means we got an OOM or similar creating a thread, or that
        // the pool and its queue are full
        getLog().error(sm.getString("endpoint.process.fail"), t);
        return false;
    }
    return true;
}
```



## 高并发下性能调优思路

* 增大任务线程池核心线程参数

* 调整最大连接数
* maxThreads调整服务器可以同时响应的客户请求数
* 