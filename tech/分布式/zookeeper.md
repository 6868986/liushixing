### zookeeper介绍

zookeeper是一个为大型分布式系统提供高可用、高性能的分布式协调服务

Kafka（MQ）、Dubbo（RPC）等都使用了zookeeper来实现分布式高可用服务。

### zookeeper特性

1. 高可用（HA）：故障发现&故障转移
2. 高性能：数据存储在内存
3. 顺序一致性：原子广播
4. 原子性：基于分布式事务

### zookeeper基本概念

#### zookeeper集群

zookeeper集群是一个基于主从复制的节点集群，节点分为三种：

1. leader：负责写操作并广播给follower
2. follower：可以接收客户端的读请求，写请求转发给leader；具有投票权
3. observer：无法投票，但是也能接收读请求&转发写请求

#### 数据模型--树形结构的文件系统***DataTree***

![一个ZooKeeper的分层命名空间](https://upload-images.jianshu.io/upload_images/4366140-b29e8378668b0441.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**数据寄存器znode**

1. 根节点 /

2. 路径均为绝对路径，zookeeper只识别绝对路径

3. 节点可以拥有数据以及子节点；数据以字节的形式存储，大小不超过1MB（基于内存）

4. 每个节点维护一个stat数据结构，用于存储该节点的状态信息

   ![img](https://static001.geekbang.org/infoq/01/01baa901dd9e3f8ed111834e81a4fd28.webp)

5. 每个节点都有一个与之关联的ACL列表（权限控制），决定谁可以对该znode进行何种操作。

   五种权限：

   - CREATE
   - READ
   - DELETE
   - WRITE
   - ADMIN

6. znode类型（创建时设置）

   - persistent：在zookeeper的命名空间中仅有一个生命周期，直至被客户端delete（客户端可以调用zookeeper的api对znode进行操作）
   - ephemeral：与创建者客户端会话绑定，会话结束 => znode消失（因此不允许有子节点），对其他客户端也是可见的（通过ACL进行配置）

7. sequential znode

#### zookeeper watch

1. 客户端与服务端保持连接

   - 轮询/pull模型
   - push模型

2. zookeeper推送通知过程

   1. 客户端在zookeeper发起读请求，传入watcher对象，注册回调函数

   2. 读请求是读DataTree上某个路径下的znode的数据，自然也就是watch这个znode

      ***疑问：***或者读某个路径？然后watch路径下面的增、删、改？

      ***补充：***watch的是什么？

      - znode数据的更改
      - znode子节点的更改
      - znode自身的创建和删除

   3. zookeeper收到对该znode的写请求，触发回调函数

   4. WatchManager通知客户端

3. zookeeper的watch是一个***一次性触发器***，client收到一次通知后必须再次设置另一个watch，这样才能知道未来的变化

   这样带来一个***问题***：client珊珊本来watch了一个znode星星，然后znode星星的数据被修改，然后zookeeper服务端发给client珊珊一个通知，然后珊珊还想watch星星，于是就需要对znode星星重新设置一个watch，那如果在收到通知之后&&设置新watch之前，其他client对znode星星进行了更改，那珊珊就感知不到了，从而丢失这次znode的变化

4. 

#### zookeeper会话

1. client与zk之间的连接本质上是一个***TCP长连接***；会话的生命周期随着连接的建立而开始，之后的request、response以及心跳机制都是通过会话实现的

   TCP连接：网络中的节点通过TCP连接，使用socket进行通信

2. zk默认端口：2181

#### zookeeper事务

#### zookeeper读写

