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

1. zookeeper

   

2. ZAB协议介绍


   Zookeeper Automic Broadcast（zookeeper原子广播协议）

3. 消息广播

4. 崩溃恢复

   zookeeper

5. 

























