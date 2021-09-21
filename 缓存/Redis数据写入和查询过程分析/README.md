

## Redis数据类型

> Redis一共有五种数据类型，分别是String（字符串），List（链表），HashMap（哈希），Set（集合），SortedSet（有序集合），HyperLogLog（统计），GEO（位置空间），Pub/Sub(发布订阅)



每种数据类型分别对应不同的底层数据结构

* String

  > 

* List

  > 

* HashMap

  > 

* Set

  > 

* Sorted Set

  >  达到

* GEO

  > 存储位置信息，可做范围查找

* HyperLogLog

  > 做统计用

* Pub/Sub

  > 发布订阅，是一种多播模式

## 全局哈希表

> Redis是一个键值对数据库，所有的键值对都保存在一个哈希表，这个哈希表也就是全局哈希表或者字典表

结构：数组+桶(链表) 



![](../静态文件/Redis底层结构.png)

## 渐进式ReHash过程

> rehash过程中需要避免阻塞







## 底层结构











