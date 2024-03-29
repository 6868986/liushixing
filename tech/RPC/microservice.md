# 微服务

## 入门

### 单体应用存在的问题

1. 部署效率低下
2. 团队协作开发成本较高
3. 系统HA差
4. 线上发布慢

### 何为服务化与微服务？

服务化：将传统的单机应用中通过JAR包依赖产生的本地方法调用，改造为通过RPC接口产生的远程方法调用。各个模块独立成一个服务部署，以RPC接口的形式对外提供服务。

微服务：

1. 服务的拆分粒度更细
2. 服务独立部署、独立维护
3. 服务治理能力要求高（统一的服务治理平台）

微服务带来的复杂度提升：

1. 服务的定义
2. 服务的发布和订阅
3. 服务的监控
4. 服务的治理
5. 故障的定位

### 服务化的拆分

1. 纵向拆分：按照业务的关联程度
2. 横向拆分：按照是否有公共部分被其他多个上游服务调用

### 微服务架构组件

1. 服务发布

   对外提供的服务名、request、response、通信协议...

   - RESTful API：常用于http协议的服务描述（swagger）
   - XML配置：常用于RPC协议的服务描述（Dubbo）
   - IDL文件：thrift、gRPC跨语言服务调用框架

2. 注册中心

   服务提供者server、服务消费者client

   1. server在注册中心上注册自己的服务**（写zk）**
   2. client向注册中心订阅自己想调用的服务**（watch node）**
   3. 注册中心返回server的地址列表给client**（读zk）**
   4. client从本地缓存的节点列表中，给予负载均衡算法选择一个并发起调用
   5. server发生变更（新增/销毁节点），注册中心将变更通知client**（watch的回调）**

   zookeeper完美契合注册中心的功能

   - 注册中心API

     1. 服务注册接口
     2. 服务销毁接口
     3. 服务订阅接口
     4. 服务变更查询接口
     5. 心跳汇报接口
     6. 服务查询接口
     7. 服务修改接口

   - 集群部署

     保证注册中心HA，通过分布式一致性协议来确保集群中节点的数据一致性

   - 目录存储

     znode树形文件结构

   - 服务健康状态监测

     session + ping

   - 服务状态变更通知

     watch

   - 白名单机制

3. 服务框架

   服务调用双方的“约定”

   - client和server如何建立连接
   - server如何处理请求：同步/异步、单连接传输/多路复用
   - 服务通信协议：TCP、UDP、HTTP？
   - 序列化方式

4. 服务监控【发现问题】

   工作原理：

   1. **数据收集**：每次服务调用的请求耗时、成功失败情况

      数据收集方式

      1. 服务主动上报：通过在业务代码或者服务框架里加入数据收集代码逻辑，在每一次服务调用完成后，主动上报服务的调用信息。
      2. 代理收集：通过服务调用后把调用的详细信息记录到本地日志文件中，然后再通过代理去解析本地日志文件，然后再上报服务的调用信息。

      核心：数据采样率 => 采集数据的频率

      ​		采样率决定了监控的实时性与精确度，采样率越高，监控的实时性就越高，精确度也越高。但采样对系统本身的性能也会有一定的影响，尤其是采集后的数据需要写到本地磁盘的时候，过高的采样率会导致系统写入磁盘的I/O过高，进而会影响到正常的服务调用。所以设置合理的采用率是数据采集的关键，最好是可以动态控制采样率，在系统比较空闲的时候加大采样率，追求监控的实时性与精确度；在系统负载比较高的时候减小采样率，追求监控的可用性与系统的稳定性。

   2. **数据传输**：

      常用方式：

      1. UDP
      2. Kafka：数据采集后发到topic，数据处理单元订阅该topic

      数据格式：

      1. 二进制协议
      2. 文本协议

   3. **数据处理**：对原始收集数据进行聚合并存储

      聚合维度：

      1. 接口：每个接口的QPS、AvgTime、成功率
      2. 机器：单节点上每个接口的QPS、AvgTime、成功率

      持久化：

      1. 索引数据库ES
      2. 时序数据库OpenTSDB

   4. **数据展示**：dashboard

   监控对象：

   1. C端功能监控
   2. 接口监控：功能所依赖的接口
   3. 资源监控：接口实现所依赖的资源（MySQL、Redis）
   4. 基础监控：服务器机器本身的健康状况，如CPU利用率、内存使用量、IO读写量、网卡带宽

   监控指标：

   1. 请求量：实时——QPS，统计——PV（page view）

   2. 响应时间：

      调用的平均耗时；

      慢请求量：请求耗时>500ms的请求数

      P99：P99 = 500ms，99%的请求响应时间在500ms以内，代表了请求的服务质量，即SLA

   3. 错误率：调用失败次数 / 调用总次数

   监控维度：

   1. 全局维度
   2. 分机房
   3. 单机
   4. 时间维度
   5. 业务核心程度维度

5. 服务追踪【定位问题】

   工作原理【调用链】mtrace

   1. traceId：唯一标识某一请求
   2. spanId：区分系统不同服务之间调用的先后关系
   3. annotation：通过代码埋点的方式，上传业务相关的数据
   4. 数据采集层：通过代码埋点收集数据并发送到mafka
   5. 数据处理层：对数据进行按需计算，然后存储供查询使用
   6. 数据展示层：

   作用：

   1. 定位并优化系统瓶颈

   2. 优化链路调用

      比如调用的下游服务a部署在了北京机房和上海机房【异地容灾】，那北京的上游业务线如果调用上海机房的a服务，势必会产生较大的网络延迟

   3. 生成网络拓扑

   4. 透明传输数据

6. 服务治理【解决问题】

   RPC调用过程中，server、网络、注册中心这三者如果有问题，就会造成调用失败

   服务治理考虑的维度：节点健康状态、节点访问优先级、调用的健康状态

   服务治理手段：

   1. 节点管理【存活探测机制】

      - 注册中心主动摘除机制

        解决server节点异常问题，由心跳机制实现【server宕机、进程异常退出...】

        带来的问题：如果是zk与server之间的网络出现了问题，这样会造成zk上注册的server节点全部被摘除，但其实这些server节点都是没问题的

      - client摘除机制

        client发起调用失败后，把本地保存的可用server列表中本次请求的server删掉

   2. 负载均衡

      1. 随机算法

      2. 轮询算法

         给server分配不同的权重，client根据不同的权重对可用节点进行轮询

      3. 最少活跃调用算法

      4. **一致性hash算法**

   3. 服务路由

      负载均衡算法 + 路由规则  =>  client调用哪个server节点

      路由规则存在原因：

      1. 业务灰度发布的需求
      2. 多机房就近访问的需求

      如何配置：

      1. 静态配置

         client本地配置

      2. 动态配置

         注册中心配置，client定期去zk同步最新的路由规则

   4. 服务容错

      不同策略：

      1. FailOver：失败自动切换

         服务消费者发现调用失败或者超时后，自动从可用的服务节点列表总选择下一个节点重新发起调用，也可以设置重试的次数。这种策略要求服务调用的操作必须是幂等的，也就是说无论调用多少次，只要是同一个调用，返回的结果都是相同的，一般适合服务调用是读请求的场景。

      2. FailCache：失败缓存

         服务消费者调用失败或者超时后，不立即发起重试，而是隔一段时间后再次尝试发起调用。比如后端服务可能一段时间内都有问题，如果立即发起重试，可能会加剧问题，反而不利于后端服务的恢复。如果隔一段时间待后端节点恢复后，再次发起调用效果会更好。

      3. FailBack：失败通知

         服务消费者调用失败或者超时后，不再重试，而是根据失败的详细信息，来决定后续的执行策略。比如对于非幂等的调用场景，如果调用失败后，不能简单地重试，而是应该查询服务端的状态，看调用到底是否实际生效，如果已经生效了就不能再重试了；如果没有生效可以再发起一次调用。

      4. FailFast：快速失败

         服务消费者调用一次失败后，不再重试。实际在业务执行时，一般非核心业务的调用，会采用快速失败策略，调用失败后一般就记录下失败日志就返回了。



























