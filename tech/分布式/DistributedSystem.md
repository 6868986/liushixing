## Distributed System

### 分布式共识算法

#### Paxos算法

1. 算法背景

   一个叫做Paxos的希腊城邦投票决议的过程；

   用来解决分布式系统中的数据一致性问题，即distributed system中各个节点如何就某个决议达成一致

2. 算法详解

   - 基本概念

     1. Proposer：提出议案Proposal = ID + value
     2. Acceptors：决策者
     3. Learner：不参与决策，从Proposer & Acceptors学习最新达成一致的提案proposal（value）

   - 基于***共识***的数据一致性 => 保证2F + 1的容错能力，即2F + 1个节点的系统最多允许F个节点同时出现故障

   - 算法流程

     ![img](https://pic2.zhimg.com/v2-a6cd35d4045134b703f9d125b1ce9671_r.jpg)

     1. Prepare：Proposer生成一个全局唯一Proposal ID【记为x】（时间戳+server ID），并向所有Acceptors广播这个Prepare(x)请求

     2. Promise：Acceptor收到Prepare请求后，做出promise承诺，包括“两个承诺、一个应答”

        承诺：不接受Proposal ID <= x的Prepare请求；不接受Proposal ID < x的Propose请求

        应答：不违背以前承诺的前提下，回复***已经Accept过的***提案中Proposal ID最大的那个提案的Value和Proposal ID，没有则返回空值。

        Acceptor对x和minProposalID进行比较。如果x > minProposalID，则minProposalID = x；然后将maxProposalID对应的id & value返回，如果没有的话则返回空值。其中，maxProposalID为已经accept过的propose中最大的ProposalID

     1. Propose：
     2. Accept：
     3. Learn：

   - 

#### Multi-Paxos算法

#### Raft算法

#### ZAB算法

1. zookeeper介绍

   https://github.com/6868986/liushixing/blob/main/knowledge/%E5%88%86%E5%B8%83%E5%BC%8F/zookeeper.md

2. ZAB协议介绍

   Zookeeper Automic Broadcast（zookeeper原子广播协议）

3. 消息广播

4. 崩溃恢复





### 分布式原理

- CAP定理

  Consistency：强一致性 > 最终一致性 > 弱一致性，

  Available：有限时间内 && 返回正常结果

  **Partition Tolerance**：分布式系统在遇到任何网络分区故障时，仍然需要能够保证对外提供满足一致性和可用性的服务，除非是整个网络环境都发生了故障。网络分区，是指分布式系统中，不同的节点分布在不同的子网络（机房/异地网络）中，由于一些特殊的原因导致这些子网络之间出现网络不连通的状态，但各个子网络的内部网络是正常的，从而导致整个系统的网络环境被切分成了若干孤立的区域。组成一个分布式系统的每个节点的加入与退出都可以看做是一个特殊的网络分区。
  方案：牺牲强一致性来换取高可用性，**P => 可扩展**

  CP => 刚性事务：2PC（XA协议）缺点：**事务阻塞型，长时间锁定资源**

  AP => BASE定理：Basic Available、Soft state、Eventually consistency => 柔性事务

- ###### 事务消息实现柔性事务

  **抢购场景**

  - 创建订单

    - 第一次租赁
      1. 锁定库存（防止超卖）（操作商品表）
      2. 创建订单、修改订单状态为1（操作订单表）
      3. 删购物车（删redis）
    - 续租（根据订单的state判断）
      1. 创建订单、修改订单状态为1（操作订单表）

  - 支付成功

    1. 修改订单状态位为2（操作订单表）
    2. 扣库存（操作商品表）
    3. 通知发货
    4. 增加积分

  - 快递员接单（**幂等**）

    1. 检查订单状态位为2
    2. 修改订单状态位为3

  - 用户确认送达

    1. 修改订单状态位为4

  - 租期满

    - 续租
      1. 直接操作订单表创单
    - 结束租赁
      1. 修改订单状态为6

  - 买家寄回

    

  - 商家收到

    1. 修改订单状态为7
    2. 增加库存



















