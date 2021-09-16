## sleep()

> 使当前线程进行睡眠模式，不释放锁

## join()

> 主要作用：使并发线程同步进行，释放锁

```java
Thread thread1 = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("线程1执行");     
    }
});

Thread thread2 = new Thread(new Runnable() {
    @SneakyThrows
    @Override
    public void run() {
        thread1.join();
        System.out.println("线程2执行");
    }
});
```

> join方法实际上引用了wait()方法，**个人理解此处会释放锁**
>
> 使当前线程thread2阻塞，等待线程thread1执行完毕之后再继续执行



## yield()

> 做出让步，使当前线程从RUNNING状态到READY状态，等待CPU重新释放时间片
>
> 当线程进入READY状态后有可能又分配到时间片继续执行
>
> 个人理解：**在多线程并发场景中，使一些不重要的线程做出让步，使重要的线程有机会分到时间片执行**



## wait()&notifyAll()&notify()

> 三个方法都是Object类的方法
>
> wait()：当前线程释放锁，并进入等待线程池中等待被唤醒
>
> notify()：随机唤醒当前锁上被等待的线程
>
>  notifyAll()：唤醒当前锁上所有被等待的线程

```java
Object object = new Object();
List<Integer> list = new ArrayList<>();
new Thread(new Runnable() {
    @SneakyThrows
    @Override
    public void run() {
        synchronized (object){
            if(list.size() == 0){
                //当前线程等待，并释放当前线程的锁
                System.out.println("线程1进入等待状态");
                object.wait();
                System.out.println("线程1被唤醒");
            }
        }
    }
}).start();

new Thread(new Runnable() {
    @Override
    public void run() {
        synchronized (object){
            System.out.println("线程2增加list数据......");
            for (int i = 0; i < 10; i++) {
                list.add(i);
            }
            object.notifyAll();
            System.out.println("线程2增加完毕l，唤醒所有等待线程......");
        }
    }
}).start();
```



* 个人理解

> 此时当前线程虽然释放了锁，但是锁上面应该记录了当前线程的信息
>
> 当当前锁调用notifyAll()唤醒方法时，会唤醒锁上面所有得等待线程

## 

* LockSupprt

  > 针对上面等待唤醒机制中无法唤醒指定线程，JDK提供了LockSuport类，可支持唤醒指定类