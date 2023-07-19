## 交换机类型

1. direct   routing key == binding key，处理有优先级的任务，指派更多的资源去消费高优先级队列
2. fanout  广播，速度最快
3. topic   routing key 和binding key 都是以 "." 分割的字符串，binding key中可以用  * # 进行模糊匹配
4. headers  Exchange与Queue绑定时指定一组键值对，取消息中的headers（也是一个键值对），对比二者是否完全匹配

## 消息丢失

1.生产者 --> RabbitMQ

事务机制：发送一条消息后使发送端阻塞，直到Broker给生产者回复

生产者确认机制： 信道channel设置为confirm模式

​								Exchange交换机成功收到一条消息后，向生产者发送ack

​								生产者收到一条/多条ack、nack后调用回调方法，根据结果对消息进行处理（记录日志，重新发送）

2.RabbitMQ 存储的消息丢失

路由不可达消息： Return消息机制 ： ReturnCallBack函数，消息从交换机路由到queue失败后调用这个函数（mandatory = true)

​								备份交换机： 将没有路由到队列中的消息发送到备份交换机上(alternate-exchange)

3.RabbitMQ --> 消费者

消费者手动确认：消费者成功消费一条消息后向broker发送ack

RabbitMQ服务异常导致重启

消息持久化：

- Message： delivery-mode = 2

- Queue

- Exchange

  当Exchange成功收到一条消息后，Rabbit会先将消息写入持久化日志，然后向生产者发送ack；消费者成功消费一条消息后，将其从持久化日志中移除。

镜像队列

## 重复消费

根源：ack网络波动  Exchange ->生产者 / 消费者->MQ broker

解决：生产消息时让其携带一个全局唯一的ID，消费时先检查是否被消费过，保证消费逻辑的幂等性 z00

## 死信队列

DLX : Dead-Letter-Exchange

消息消费失败后进入DLX,然后进入死信队列

消费失败原因：消息被拒&没有二次入队（requeue = false）

​							消息超时(ttl)未消费

​							达到最大队列长度

## 路由

1. 消息生产时的routing key
2. 队列绑定到交换机时指定的binding key（fanout交换机类型无视binding key）
3. 根据交换机类型不同，将routing key 与 binding key进行匹配

多个消费者可以订阅同一个队列，队列中的消息会被平均分摊给多个消费者（Round-Robin轮询）

## 消息传输

TCP连接的创建&销毁开销大，且并发数受系统资源限制；RabbitMQ使用信道（channel）来传输数据。

信道是建立在真实TCP连接中的虚拟连接，且每条TCP连接上的信道数量没有限制。

一条TCP连接被多个线程共享，一个信道对应一个线程。

## 高可用

​	1.普通集群

​	2.镜像模式

​	镜像队列的master、slave节点是相对于某个队列而言的。不是⼀个node作为所有queue的master，其它node作为slave。⼀个queue第⼀次创建的node为它的master节点，其它node为slave节点。

⽆论客户端的请求打到master还是slave最终数据都是从**master节点**获取。当请求打到master节点时，master节点直接将消息返回给client，同时master节点会通过GM（Guaranteed Multicast）协议将queue的最新状态⼴播到slave节点。GM保证了⼴播消息的**原⼦性**，即要么都更新要么都不更新。

当请求打到slave节点时，slave节点需要将请求先重定向到master节点，master节点将将消息返回给client，同时master节点会通过GM协议将queue的最新状态⼴播到slave节点

如果有新节点加⼊，**RabbitMQ不会同步之前的历史数据**，**新节点只会复制该节点加⼊到集群之后新增的消息。**

- **从节点宕机**    当slave宕掉了，除了与slave相连的客户端连接全部断开之外，没有其他影响。

- **主节点宕机**

  当master宕掉时，会有以下连锁反应：

  1. 与master相连的客户端连接全部断开；
  2.选举最老的slave节点为master（因为最老的slave与前任master之间的同步状态应该是最好的）。若此时所有slave处于未完全同步状态，则未同步部分消息丢失；
  3.新的master节点requeue所有unack消息，因为这个新节点无法区分这些unack消息是否已经到达客户端，亦或是ack消息丢失在老的master的链路上，亦或者是丢在master组播ack消息到所有slave的链路上。所以处于消息可靠性的考虑，requeue所有unack的消息。此时客户端可能有重复消息；
  4.如果客户端连着slave，并且Basic.Consume消费时指定了x-cancel-on-ha-failover参数，那么客户端会受到一个Consumer Cancellation Notification通知。如果未指定x-cancal-on-ha-failover参数，那么消费者就无法感知master宕机，会一直等待下去。

元数据 + queue中的message 存在于多个mq实例

## 持久化

1. 交换机
2. 队列 （元数据 + 消息数据）队列持久化保证队列元数据不丢失
3. 消息

Exchange  Queue 都是默认持久化的，Message的持久化是代码层面的，与delivery-mode有关

将所有消息都设置为持久化，会严重影响rabbitmq服务器性能，写入磁盘的速度比写入内存的速度慢得不只一点点。所以对于可靠性不是那么高的消息可以不采用持久化处理以提高整体的吞吐量。在选择是否要将消息持久化时，需要在可靠性和吐吞量之间做一个权衡

三者均开启持久化并不能完全保证消息不丢失

1. ​	消费者  autoAck = false

2.    持久化的消息存入到mq之后，还需要一小段时间才能存入磁盘

   ​	如果在这段时间内rabbitmq服务节点发生了宕机、重启等异常情况，消息保存还没来得及落盘，那么这些消息将会丢失。这种情况可以使用镜像队列来解决。

## 存储机制

1. 存储方式

   消息的持久化 -> 在“持久层”中完成

   持久层：

   1. 队列索引（queue_index)  .idx文件

      ​	存储队列中落盘的消息的信息 ，一个队列对应一个queue_index

   2. 消息存储（msg_store）.rdq文件

      ​	以键值对的形式存储消息，一个mq节点有一个msg_store

      - ​	msg_store_persistent ：负责持久化消息的持久化 

        ​	rabbitmq启动后，会针对每个vhost启动两个进程，该进程负责将持久化消息（delivery-mode = 2）写入文件与从文件中读取消息

      - ​	msg_store_transient

   - 若消息直接存在队列索引中，则当消息通过exchange同时路由到多个队列时，此消息会被写到每个队列的索引文件中。
   - 若消息是存在消息存储中，就仅仅只有一个副本。

2. 存储文件

   - 队列索引 0.idx  1.rdx ...  段文件

   - 消息存储 0.rdq 1.rdq ...   经过 rabbit_msg_store 处理的所有消息都会以追加的方式写入到文件中，当一个文件的大小超过指定的限制 (file_size_lmit)后，关闭这个文件再创建一个新的文件以供新的消息写入，文件后缀是“ .rdq ”。文件名从0开始进行累加，所以文件名最小的文件也是最老的文件。
   - 垃圾回收文件合并机制

3. 存储原理

   - 生产者消息写入原理
   - 消费者消息读取原理

4. ETS

5. 队列结构

   queue = 负责协议相关的消息处理 + 消息存储的具体形式和引擎

   - 接收生产者发布的消息、向消费者交付消息、处理消息的确认 (包括生产端的 confirm 和消费端的 ack) 等
   - 

