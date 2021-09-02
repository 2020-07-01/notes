

> 最近在项目中需要开启多个线程去构建数据，但又考虑前端页面超时，因此后台使用了CountDownLatch
>
> 常见的同步工具还包括Semaphore，



底层实现：

> **三种同步工具底层实际运用了AQS同步框架**



## CountDownLatch

> CountDownLatch是一个同步工具，允许单个或者多个线程等待直到所有线程操作都完成，主线程才继续向下执行

* **注意**

  > **await()方法可能会造成阻塞**
  >
  > 建议使用await(long timeout, TimeUnit unit)方法

### 源码解析

```java
public class CountDownLatch {
   	//实际使用了AQS
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            //AQS 的state
            setState(count);
        }

        int getCount() {
            return getState();
        }

        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }

        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }

    private final Sync sync;

   	//构造方法
    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }

    //导致当前线程等待直接计数器为0，或者当前线程被中断
    //当前线程为主线程，主线程等待
    //如果当前计数器为0，此方法直接返回
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
    
	//导致当前线程等待，直到计数器为0，或者时间到达，或者当前线程被中断
    //如果计数器为0 返回true
  	//如果等待超时，返回false，等待超时之后，此方法不会等待
    public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }
	//计数器减一
    public void countDown() {
        sync.releaseShared(1);
    }

    public long getCount() {
        return sync.getCount();
    }
    
    public String toString() {
        return super.toString() + "[Count = " + sync.getCount() + "]";
    }
}
```



## Semaphore

> 信号量机制，通过设置信号量来限制某些资源的访问数，通过协调各个线程来保证合理的使用资源
>
> **限流**
>
> RabbimtMQ源码中有用到

### 源码分析

* **Semaphore分为公平信号量和非公平信号量**

```java
//底层实现AQS
public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

* 获取许可证，可中断

  > 在未获取到许可证之前一致处于阻塞状态，除非当前线程被中断
  >
  > ```java
  > public void acquire() throws InterruptedException {
  >     sync.acquireSharedInterruptibly(1);
  > }
  > 
  > public void acquire(int permits) throws InterruptedException {
  >     if (permits < 0) throw new IllegalArgumentException();
  >     sync.acquireSharedInterruptibly(permits);
  > }
  > ```

* 获取许可证，不可中断

  > 在未获取到许可证之前一致处于阻塞状态，不可中断
  >
  > ```java
  > public void acquireUninterruptibly() {
  >     sync.acquireShared(1);
  > }
  > public void acquireUninterruptibly(int permits) {
  >     if (permits < 0) throw new IllegalArgumentException();
  >     sync.acquireShared(permits);
  > }
  > ```

* 是否是公平锁

  > ```java
  > public boolean isFair() {
  >     return sync instanceof FairSync;
  > }
  > ```

* 释放许可证

  > **注意点：释放许可证的线程不一定是获取许可证的线程**
  >
  > ```java
  > public void release() {
  > 	sync.releaseShared(1);
  > }
  > public void release(int permits) {
  > 	if (permits < 0) throw new IllegalArgumentException();
  > 	sync.releaseShared(permits);
  > }
  > ```

* **获取剩余许可证**

  >```
  >semaphore.availablePermits() //直接放回
  >semaphore.drainPermits() //先获取再返回
  >```
  >
  >```java
  >//返回当前可用的许可证
  >public int availablePermits() {
  >    return sync.getPermits();
  >}
  >
  >//获取并返回
  >final int drainPermits() {
  >    for (;;) {
  >        int current = getState();
  >        if (current == 0 || compareAndSetState(current, 0))
  >            return current;
  >    }
  >}
  >```

### acquire()方法获取许可证时抛出异常踩坑点

> 首先，通过阅读源码得知
>
> 1.acquire方法在未获取到信号量之前一直处于阻塞状态，或者被中断

下面是业务中部分代码：

```java
try {
    if (maxChannel > 0) {
        semaphore.acquire();
    }
    //执行业务逻辑......
} catch (Exception e) {
    semaphore.release();
    log.error("send message error", e);
}
```

> 流程：当线程未获取到许可证时，抛出了中断异常，此时会释放许可证
>
> 问题：**从acquire方法注释得知，当抛出中断异常时，当前线程会自动释放许可证**
>
> 而上面代码的catch块中手动释放了许可证，会导致许可证的数量变多，应该对中断异常进行特殊处理



## 场景分析

* CountDownLatch

  > 适用于对任务进行拆分，使其多线程并行执行，执行完成之后再由主线程对其结果进行汇总

* Semaphore

  > 通过对资源设置许可证控制资源的访问数，以合理使用资源