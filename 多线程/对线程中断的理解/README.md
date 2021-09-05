> 之前在研究线程同步框架Semaphore时注意到，当线程发生中断异常时，会自动释放许可证，特此深入研究一下线程中断
>
> 线程是CPU调度的最小单位



## 对中断操作的理解

> 首先需要明白一个概念，Thread提供了中断方法**intrrupt()**，每个线程都有一个布尔类型的属性，表示线程的中断状态，当调用中断方法时，会设置这个中断状态
>
> **此时表示当前线程是可中断的，但并不会强制此线程停止工作，仅此更新中断状态而已**
>
> Thread提供了一个方法用来判断当前线程是否是可中断的-isInterrupted()
>
> 当线程为可中断时，可根据线程是否可中断来进行一些其他操作，比如停止或阻塞此线程，抛出中断异常



* API

  ```java
  Thread thread = new Thread();
  //中断线程，设置中断状态
  thread.interrupt();
  
  //判断线程是否可中断的，不清除中断位
  thread.isInterrupted();
  
  //判断线程是否可中断的，清除中断位
  thread.interrupted();
  ```

* 对比aqs源码进行理解

```java

public final void acquireSharedInterruptibly(int arg) throws InterruptedException {
	//如果线程是中断的，则抛出异常
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}

private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                //如果线程是中断的，则抛出异常
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

## 中断敏感性

> 自此先复习一下线程的六种状态
>
> 1.NEW	新建状态
>
> 2.RUNNABLE	可运行状态 分为RUNNING 和 READY
>
> 3.BLOCKED	阻塞状态
>
> 4.WAITING	等待状态
>
> 5.TIME_WAITING	超时等待状态
>
> 6.TERMINATED	终止状态

* **当线程处于BLOCKED和RUNNABLE状态时，对中断操作是不敏感的，只是设置线程中断状态，具体操作交给程序员来处理**

* **当线程处于WAITING 和 TIME_WAITING时，对于线程的中断操作会抛出中断异常**

```java
Thread threadA = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("线程A启动......");
        try {
            System.out.println("线程A睡眠10秒......");
            //线程A睡眠10秒
            Thread.currentThread().sleep(10 * 1000);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
});
threadA.start();

Thread threadB = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("线程B启动......");
        System.out.println("线程A中断......");
        //当线程A中断时，会抛出中断异常
        threadA.interrupt();
    }
});
threadB.start();
```



## 如何安全的结束一个线程

1. 使用标志状态
2. 使用中断方法



