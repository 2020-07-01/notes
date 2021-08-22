



> 前段时间对mq进行改造，进行系统数据流通变更，现结合自己知识体系做一个总结

**业务场景**

> 中心2.0数据变更->canal监听->MQ->中心sync-listener监听->中心3.0&&分省MQ->分省sync-listener监听mq->入库&&数据变更mq
>
> 中心和分省需同步的表相同

* 中心交换机和分省交换机：topic，同一套代码，不同的队列监听
* 数据变更交换机：front，数据变更之后通知给其他业务线

# Exchange

> 交换机，根据规则将消息路由到指定的队列

* fanout

  >广播交换机，忽略rounting key的存在，将消息路由到绑定的所有队列中

* direct 

  >直连交换机，mq默认的交换机，完全根据rounting key路由消息，**精确匹配**
  >
  >mq默认的交换机为一个名称为空的交换机

* topic

  >主题交换机，根据rounting key路由消息，rounting key支持模糊匹配

* headers

  >待实践

# 解决方案

## 生产端消息可靠性投递

* 入库打标记

  ![可靠性投递方案](D:\workspace\github\notes\中间件\静态资源\可靠性投递方案1.png)





* 延迟投递











参考文章：

https://www.huaweicloud.com/articles/6a2d97cb2cb620b934187ff2670b62f5.html