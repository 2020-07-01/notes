> 三个月前搭建了研发环境的Redis集群，中间过程踩了不少坑；上周由于停电，Redis集群挂了，不能自动启动，至此对Redis集群进行梳理
>
> 研发环境的Redis集群三个节点 node1,node2,node3

关于节点动态删除，槽位变化，数据指定节点等后续在研究整理（待研究）

下面记录一下搭建过程

## 相关配置

* 持久化方式

  > 研发环境采用RDB快照方式
  >
  > ```json
  > save 900 1 # 每900秒至少有一个键值改动则进行快照
  > save 300 10
  > save 60 10000
  > ```

* cluster-enabled 

  > cluster-enabled yes  开启redis集群 
  >
  > 节点启动有方式是集群方式

* cluster-node-timeout

  > cluster-node-timeout 5000 节点超时限制
  >
  > 当某个节点向其他节点发送ping命令，但是目标节点未在限制时间内回复，则认为此节点可能已失效

* lua-time-limit

  > lua-time-limit 5000 设置lua脚本的最大运行时间

* timeout

  > timeout 0  客户端空闲N秒后断开连接  0表示不启用

* tcp-keepalive

  > tcp-keepalive 300 值非0的情况下表示服务端周期性的检测客户端是否可用
  >
  > 此处涉及**TCP长连接和短链接之分**

* daemonize

  > daemonize yes 启用后台守护进程运行模式

* pidfile

  > /usr/local/redis/redis-7003/redis_7003.pid  redis启动后的进程ID文件

* cluster-config-file

  > 指定集群配置文件

* masterauth

  > masterauth passwd123
  >
  > 集群节点之间相互通信的密码

* hz

  > hz 10  每秒运行10次检查是否过期
  >
  > 此处**涉及key的过期删除策略**
  >
  > 通过内部函数开启后台任务清除过期key等等

* Redis 集群模式下只支持db0，不支持db的选择

## 搭建步骤

* 第一步：启动节点

  > redis-server  nodes-7005.conf
  >
  > 切记：此命令应该在bin目录执行，否则无发启动redis
  >
  > bin 目录下有redis-cli、redis-server等

* 第二步：启动Redis 集群

  > redis-cli --cluster create ip:7003 ip:7004 ip:7005  -a 密码

## redis-cli

Redis Cluster在5.0版本之后取消了ruby脚本的支持，集成到了redis-cli中，极大方便使用。

> 使用redis-cli --cluster help 查看该命令的具体使用方式

