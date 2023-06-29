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

   - 轮询/pull模型（client向server）
   - push模型（server向client）

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

   ​		这样带来一个***问题***：client珊珊本来watch了一个znode星星，然后znode星星的数据被修改，然后zookeeper服务端发给client珊珊一个通知，然后珊珊还想watch星星，于是就需要对znode星星重新设置一个watch，那如果在收到通知之后&&设置新watch之前，其他client对znode星星进行了更改，那珊珊就感知不到了，从而丢失这次znode的变化

4. 

#### zookeeper会话

1. 会话简介

   ​		client与zk之间的连接本质上是一个***TCP长连接***；会话的生命周期随着连接的建立而开始，之后的request、response以及心跳机制（ping）、watch通知都是通过会话实现的

2. zk默认端口：2181

3. Session的建立

   ​		client维护一个zk的server集群列表，启动时，client遍历这个列表并尝试连接server，失败后就尝试连接下一个server，直至连接成功。一旦连接成功，相当于在client和server之间存在了一个TCP长连接，同时server也会开始为这个client维护一个Session

4. Session的属性参数

   - sessionID：会话的唯一标识 = 时间戳 + 机器ID

     Map<Long,SessionImpl>  => 一个sessionID对应一个session的实现

   - timeout：会话的超时时间；client配置好超时时间并发送给server，然后server根据自己的超时时间的限制来确定最终的timeout

     ​		当由于网络中断、server负载过大等原因导致client与server断开连接时，这次会话以及其相关的ephemeral znode并不会立即被删除，只要在timeout时间内，client重新连接上了任何一个server，则之前的会话依旧有效。但是如果session过期了，集群中server会删除所有这个session创建的ephemeral znode

   - expiration time：绝对过期时间（时间轴上的某个时间点）

   - isClose：会话是否关闭的标识，如果为true，则zk server不会处理来自这个session的任何请求

5. Session分桶机制
           zookeeper的server会定期（***expiration interval***）对会话进行超时检测，若对Session不做处理，则每次检测都需要遍历所有Session，这肯定无法接受；

   ​		因此会按照expiration time对Session进行分桶处理，这样zk server每次超时检测只需要扫描一个对应的桶；

   ​		同时对expiration time做近似处理，让其为expiration interval的倍数；
   ​		临时绝对过期时间：expirationTime0  = current time + timeout;

   ​		***zk最终的过期时间***：expirationTime = (expirationTime0 / expirationInterval + 1) * expirationInterval;

   ​		这样可以让Session的expiration time不那么分散，避免出现n个Session有n个expiration time的情况；同时也可以让Session的过期时间与server的超时检测时间重叠，避免出现上一秒超时检测完下一秒才到绝对超时时间的情况

6. Session续约

   ​		Session的timeout并非一个定值，而是随着client和server之间的交互而不断更新，随之而来的就是Session在桶上的不断迁移；

   ​		client的读请求、写请求、ping心跳都会对session进行续约

   ​		续约逻辑：

   ​				zk server接收到client的读写请求/ping => server检查当前session是否已经关闭（isClose） => false

   ​		  => 根据current time、新的timeout、expiration interval来计算session的expiration time，记为new

   ​		  => 获取到session原来的expiration time，记为old，并根据old找到session对应的桶

   ​		 => 从桶中移除session

   ​		 => 更新session的expiration time为新计算的expiration time，即new

   ​		 => 根据new找到新的桶，并将session加入该桶，从而完成session在桶中的迁移

7. TCP连接：网络中的节点通过TCP连接，使用socket进行通信

   长连接：建立TCP连接 => socket收发完数据 => 保持TCP连接

   ​				 => socket开始新一轮的收发数据 => 保持TCP连接 => ...... => 关闭TCP连接

   短连接：建立TCP连接 => socket收发完数据 => 关闭TCP连接

   长短连接的选取：短连接需要频繁的“三握四挥”，耗费资源；长连接在高并发的场景下，存在很多未被有效使用的TCP连接，同样也会耗费资源；因此长短连接要灵活选取，没有绝对的优势

   eg：使用JDBC进行数据库连接的时候建立的数据库连接池（类似ThreadPool）

   参数1:minPoolSize    最小连接数

   参数2:maxPoolSize    最大连接数

   参数3:maxIdleTime    最大空闲时间

   其中最小连接数中的连接，即使未被使用也会一直存在 => 长连接

   非最小连接数中的连接，如果一直未被使用以至于超过了最大空闲时间，则连接会被回收 => 短连接

#### zookeeper事务

#### zookeeper读写

