> 关于JVM实践操作还不是很丰富，各种参数比较多，暂且将用到参数进行记录，避免查询参数花费时间



* -XX:+UseG1GC

  > 使用G1垃圾收集器

* -XX:+UseSerialGC

  > 使用Serial收集器

* -XX:+UseConcMarkSweepGC

  > 指定CMS垃圾收集器

* -XX:+PrintCommandLineFlags -version

  > 查看程序使用的**默认**JVM参数

* -XX:+PrintGCDetails

  > 打印GC日志
  
* -XX:+UseCMSCompactAtFullCollection

  > 当CMS收集器要进行FullGC时开启内存碎片的合并整理
  >
  > 因为CMS是基于标记-清除算法的，此算法会产生大量的内存碎片

* -XX:MaxTenuringThreshold

  > 对象晋升到老年代的年龄阈值 默认是15
  >
  > 对象每经历一次GC，年龄值+1

