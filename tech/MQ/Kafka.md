#### Kafka(Mafka)

## Kafka

#### 简介

kafka是最初由Linkedin公司开发，由Scala和Java编写，Kafka是一个分布式、分区的、多副本的、多订阅者的、基于zookeeper协调的日志系统(分布式MQ系统)，其持久化层本质上是一个“按照分布式事务日志架构的大规模发布/订阅消息队列”，可以用于web/nginx日志，搜索日志，监控日志，访问日志等等，Linkedin于2010年贡献给了Apache基金会并成为顶级开源项目。

#### 消息系统介绍

一个消息系统负责将数据从一个应用传递到另外一个应用，应用只需关注于数据，无需关注数据在两个或多个应用间是如何传递的。分布式消息传递基于可靠的消息队列，在客户端应用和消息系统之间异步传递消息。有两种主要的消息传递模式：**点对点传递模式、发布-订阅模式**。大部分的消息系统选用发布-订阅模式。**Kafka就是一种发布-订阅模式**。

- 点对点消息传递模式

  **生产者发送一条消息到queue，只有一个消费者能收到**。

  消息持久化到一个队列中。此时，将有一个或多个消费者消费队列中的数据。但是一条消息只能被消费一次。当一个消费者消费了队列中的某条数据之后，该条数据则从消息队列中删除。该模式即使有多个消费者同时消费数据，也能保证数据处理的顺序

  ![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180507190326476-771565746.png)

- 发布-订阅消息传递模式

  **发布者发送到topic的消息，只有订阅了topic的订阅者才会收到消息**。

  消息被持久化到一个topic中。与点对点消息系统不同的是，消费者可以订阅一个或多个topic，消费者可以消费该topic中所有的数据，同一条数据可以被多个消费者消费，数据被消费后不会立马删除。在发布-订阅消息系统中，消息的生产者称为发布者，消费者称为订阅者。

  ![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180507190443404-1266011458.png)

#### 设计目标

1. 消息持久化：以时间复杂度为O(1)的方式提供消息持久化能力，即使对TB级以上的数据也能保证常数时间复杂度的访问性能。
2. **高吞吐**：在廉价的商用机器上也能支持单机每秒10万条以上的吞吐量。
3. 分布式：支持消息分区以及分布式消费，并保证分区内的消息顺序。
4. 跨平台：支持不同技术平台的客户端（如Java、PHP、Python等）。
5. 实时性：支持**实时**数据处理和**离线**数据处理。
6. 伸缩性Scale out：支持水平扩展

高吞吐量是其核心设计之一。

支持其高吞吐量的设计：

1. PageCache
2. zero-copy
3. 支持数据批量发送和拉取
4. 支持数据压缩
5. Topic划分为多个partition，提高并行处理能力

#### 总体结构

![截屏2023-05-16 09.52.10](/Users/liushixing/Library/Application Support/typora-user-images/截屏2023-05-16 09.52.10.png)

#### 优点

1. ***解耦***

   在项目启动之初来预测将来项目会碰到什么需求，是极其困难的。消息系统在处理过程中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一接口。这允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

2. 冗余（Replica）

   有些情况下，处理数据的过程会失败。除非数据被持久化，否则将造成丢失。消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。

3. 扩展性

   因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的，只要另外增加处理过程即可。不需要改变代码、不需要调节参数。扩展就像调大电力按钮一样简单。

4. ***削峰***

   在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见；如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

5. 可恢复性

   系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

6. 消息有序

   在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。Kafka保证一个Partition内的消息的有序性。

7. 缓冲

   在任何重要的系统中，都会有需要不同的处理时间的元素。例如，加载一张图片比应用过滤器花费更少的时间。消息队列通过一个缓冲层来帮助任务最高效率的执行———写入队列的处理会尽可能的快速。该缓冲有助于控制和优化数据流经过系统的速度。

8. ***异步***

   很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。

#### Kafka基本概念

- 名词释义

  1. broker：一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic。
  2. topic：可以理解为消息的类型 => 逻辑概念。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）
  3. partition：为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，一个topic可以分为多个partition，每个partition是一个有序的队列。partition中的每条消息都会被分配一个有序的id（offset），每个partition中的数据使用多个segment文件存储。kafka只保证按一个partition中的顺序将消息发给consumer，不保证一个topic的整体（多个partition间）的顺序。
  4. producer：生产者即数据的发布者，该角色将消息发布到Kafka的topic中。broker接收到生产者发送的消息后，broker将该消息**追加**到当前用于追加数据的segment文件中。生产者发送的消息，存储到一个partition中，生产者也可以指定数据存储的partition。
  5. consumer：消费者可以从broker中读取数据。消费者可以消费多个topic中的数据。
  6. consumer group：若干个Consumer组成的集合。这是kafka用来实现一个topic消息的广播（发给所有的consumer）和单播（发给任意一个consumer）的手段。一个topic可以有多个CG。每个分区只会分给消费组中的一个consumer。如果需要实现广播，只要每个consumer有一个独立的CG就可以了。要实现单播只要所有的consumer在同一个CG。用CG还可以将consumer进行自由的分组而不需要多次发送消息到不同的topic。
  7. offset：Kafka中的消息都是顺序保存在磁盘上的，通过offset偏移量来标识消息的顺序
  8. replica：kafka采用分区（Partition）的方式，使得消费者能够做到并行消费，从而大大提高了自己的吞吐能力。同时为了实现高可用，每个分区又有若干份副本（Replica），这样在某个broker挂掉的情况下，数据不会丢失。
  9. leader：每个partition有多个副本，其中有且仅有一个作为Leader，Leader是当前负责数据的读写的partition。
  10. follower：Follower跟随Leader，所有写请求都通过Leader路由，数据变更会广播给所有Follower，Follower与Leader保持数据同步。如果Leader失效，则从Follower中选举出一个新的Leader。当Follower与Leader挂掉、卡住或者同步太慢，leader会把这个follower从“in sync replicas”（ISR）列表中删除，重新创建一个Follower。
  11. ISR：in sync replicas，消息同步能跟上leader partition的follower partition的集合列表，offset > HW
  12. OSR: out sync replicas
  13. HW：high watermark，所有replica中最小的offset

  

- partition模型

  ​	1、一个partition只能被同组的一个consumer消费（图中只会有一个箭头指向一个partition）

  ​	2、同一个组里的一个consumer可以消费多个partition（图中第一个consumer消费Partition 0和3）

  ​	3、消费效率最高的情况是partition和consumer数量相同。这样确保每个consumer专职负责一个partition。

  ​	4、consumer数量不能大于partition数量。由于第一点的限制，当consumer多于partition时，就会有consumer闲置。

  ​	5、consumer group可以认为是一个订阅者的集群，其中的每个consumer负责自己所消费的分区

  ​	Kafka每个主题的分区日志分布式的存储在Kafka集群上，同时为了故障容错，每个分区都会以副本的方式复制到多个消息代理节点上。其中一个节点会作为主副本（Leader），其他节点作为备份副本。

  ​	主副本会响应所有客户端读写操作，备份副本仅仅从主副本同步数据。当主副本出现故障时，备份副本中的一个副本会被选择为新的主副本。

  ​	因为每个分区的副本中只有主副本接受读写，所以每个服务端都会作为某些分区的主副本，以及另外一些分区的备份副本，这样Kafka集群的所有服务端整体上对客户端是负载均衡的。

  ​	分区是消费者线程模型的最小并行单位，采用分区后，如果有两个分区，最多两个消费者同时消费，消费的速度肯定会更快。如果觉得不够快，可以加到四个分区，让四个消费者并行消费。分区的设计大大的提升了kafka的吞吐量！

- zookeeper

  Broker 的设计是无状态的，可以水平扩展，集群的元数据都存储在zookeeper 上，在broker启动的时候会向zookeeper 上报集群信息，Broker 中的控制器（controller）和协调器利用zk 中存储的元数据来管理集群，controller 管理单个分区多副本间的选主，故障转移等（可以看做是运维层面的组件），协调器是用来协调消费者工作的分配

#### Kafka Producer

生产者提供了同步和异步两种发送接口

1. ```java
   try{
       //Mafka同步发送
       ProducerResult producerResult = producer.sendMessage("这是第" + i + "条消息");
   
       //Mafka异步发送
       String message = "这是第" + i + "条消息";
       ProducerResult producerResult1 = producer.sendAsyncMessage(message,
           new FutureCallback() {
               @Override
               public void onSuccess(AsyncProducerResult asyncProducerResult) {
                   System.out.println("发送成功" + message);
               }
   
               @Override
               public void onFailure(Throwable throwable) {
                   System.out.println("发送失败" + message);
               }
            }
        );
   } catch (Exception e) {
       throw new RuntimeException(e);
   }
   ```

   ​		当produer调用send方法，发送消息的时候，只是先把消息缓存到一个队列，由该模式的消费者（另一个线程）来执行真正的发送逻辑。这样主要是为了发送的时候尽量是批次的消息发送，而非单条单条消息的发送，用来提升发送性能。

   ​		sender是在一个异步线程（ioThread）中执行主要逻辑，不停的从accumulator消息累加器中获取准备发送的消息批次并通过网络发送到目标broker上，基本流程如下：

   ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1de3174ef044d0cb77e04c90da20a64~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

2. kafka生产者会将消息封装成一个 ProducerRecord ，再向 kafka集群中的某个 topic 发送消息

3. ProducerRecord => send()方法 => 序列化器 => 分区器 => 

4. kafka 为了提升效率，生产者发送消息时批量的，采用的是生产者消费者线程模型，在客户端每个分区一个队列，获取到分区后，每个消息放到所属分区对应队列的批记录中，kafka 的记录收集器（RecordAccumulator） 负责生产者客户端生产的消息，发送线程（Sender）负责读取记录收集器的批量消息，通过网络发送给服务端，当批记录满时，sender线程从记录收集器获取批记录是按Broker 分组的，发送线程轮询，按Broker 发送批记录。

   ![截屏2023-05-18 15.47.40](/Users/liushixing/Desktop/截屏2023-05-18 15.47.40.png)

   ​		以Batch的方式推送数据可以极大的提高处理效率，kafka Producer 可以将消息在内存中累计到一定数量后作为一个batch发送请求。Batch的数量大小可以通过Producer的参数控制，参数值可以设置为累计的消息的数量（如500条）、累计的时间间隔（如100ms）或者累计的数据大小(64KB)。通过增加batch的大小，可以减少网络请求和磁盘IO的次数，当然具体参数设置需要在效率和时效性方面做一个权衡。

   ​		Producers可以异步并行的向kafka发送消息，但是通常producer在发送完消息之后会得到一个future响应，返回的是offset值或者发送过程中遇到的错误。这其中有个非常重要的参数“acks”,这个参数决定了producer要求leader partition 收到确认的副本个数。

5. 

#### Kafka Consumer

#### Kafka Broker

###### ***leader broker***

创建、删除主题、增加分区并分配leader分区；集群broker管理，包括新增、关闭和故障处理；

分区重分配（auto.leader.rebalance.enable=true），分区leader选举

每个broker都有自己的broker id，启动后去zookeeper上竞争controller节点，抢到的即为broker leader，

###### ***Replica***

1. follower故障

   体现在参数落后时间，LEO同HW差太多，直接将其从ISR移除，直至LEO追上HW

2. leader故障

   将leader replica从ISR移除，然后从ISR中选举新的leader，然后将所有replica高于HW的消息删除，之后继续从新的leader同步消息

##### 分区平衡

时间戳索引文件：时间戳 + 相对offest，查询某时间范围内的消息

根据时间戳查出offset，然后根据offest去index文件找偏移量索引，然后去log文件找数据

###### 消费者组初始化流程

1. consumer发送joinGroup请求，groupid一致
2. 选出leader consumer
3. 

##### Coordinator

主要用于offset管理和消费者的Rebalance

- 对于一个consumer group，求_consumer_offset这个topic上的分区号，求法：groupid.hashcode() % 50,求出来的partition所在的那个broker即为coordinator



#### Kafka存储层 ---持久化

###### 













