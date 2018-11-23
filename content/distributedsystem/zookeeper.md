---
title: "zookeeper"
date: 2018-05-07 16:39
---

# Zookeeper简介
Zookeeper最早起源于Yahoo 的Hadoop项目，作为分布式计算和存储平台，需要解决一致性的场景非常多，因此他们开发了Zookeeper这么一个组件，专门用来协调解决一致性问题。

那什么才是一致性呢？数据一致性其实最早是数据库系统中的概念。我们可以简单的把一致性理解为正确性或者完整性，那么数据一致性通常指关联数据之间的逻辑关系是否正确和完整。比如你删除某个被其他表引用的表时，你是删除不了的，这个就是关系的一致性保证的。而在分布式系统中，数据一致性往往指的是由于数据的复制，不同数据节点中的数据内容是否完整并且相同。

​比如在集中式系统中，有一些关键的配置信息，可以直接保存在服务器的内存中，但是在分布式系统中，如何保存这些配置信息，又如何保证所有机器上的配置信息都保持一致，又如何保证修改一个配置能够把这次修改同步到所有机器中呢？

再举个数据库主从例子，比如你的服务请求太大了，一台数据库已经无法支撑这么大量的请求，你又一个办法就是把数据复制到多个数据库上，这样可以分担请求压力，实现采用负载均衡技术就可以实现请求压力的分散。但是问题来了，如何保证每一个数据库都有相同的数据呢？现在一般主流解决方案都是数据库主从模式，一个主库做写，另外多个从库做读使用，这也意味着上层业务属于读多写少的业务。那一个数据写入之后，就必须把数据复制到其他从库上。这样其他读请求才可以看到。

根据读请求看到新写入数据的时机，我们又把这种一致性模型分成强一致性，弱一致性和最终一致性。
 - 强一致性：就是更新之后的数据，读请求立即可以见到。
   - 要做到这个意味着，你的数据必须更新到从库之后，写才打算执行成功，才能返回。当然也不一定是非要等到所有的从库都更新完，可以更新一半以上写操作就算成功并返回，此时读操作就不能简单只读某一个从库了，得同时读一半以上的从库，根据鸽巢原理，这样必定能读到一个最新的数据，我们可以给数据更新加版本号，读出来的所有数据看那个版本最新，就取哪一个。
- 弱一致性：就是更新之后的数据，读请求不能立即看到。
  - 系统并不保证续进程或者线程的访问都会返回最新的更新过的值。系统在数据写入成功之后，不承诺立即可以读到最新写入的值，也不会具体的承诺多久之后可以读到。但会尽可能保证在某个时间级别（比如秒级别）之后，可以让数据达到一致性状态。
- 最终一致性：更新数据之后走，不保证更新数据立即课件，也不会保证多久可见，但一定在某个时间会可见。
  - 弱一致性的特定形式。系统保证在没有后续更新的前提下，系统最终返回上一次更新操作的值。在没有故障发生的前提下，不一致窗口的时间主要受通信延迟，系统负载和复制副本的个数影响

实际上一致性模型并不是这么简单的三大类，他有很多应用场景和细分类别。都是根据不同需求和应用场景来划分的。[分布式系统一致性分类，你知道几种？](https://cloud.tencent.com/developer/article/1015442). 本文不再赘述这些一致性模型，超出了范围。

# Zookeeper的重要特性
Zookeeper 提供的最常用的两个基本特性就是基于内存的文件系统操作和事件通知机制，而且这些增删改操作都是原语性质的（对所有的客户端来说）。

## Zookeeper 文件系统
数据以目录树的结构存储在内存之中，使得Zookeeper处理数据更快，可以保证高吞吐量和低延迟，但同时数据存储的瓶颈也是内存容量，因此Zookeeper 不能存储海量数据，只能存储少量或者中等规模的数据。目录树节点的存储数据上限是1M.

## Zookeeper 文件目录节点znode类型
1. **PERSISTENT-持久化目录节点**
客户端与zookeeper断开连接后，该节点依旧存在 
2. **PERSISTENT_SEQUENTIAL-持久化顺序编号目录节点**
客户端与zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号 
3. **EPHEMERAL-临时目录节点**
客户端与zookeeper断开连接后，该节点被删除 
4. **EPHEMERAL_SEQUENTIAL-临时顺序编号目录节点**
客户端与zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号

## Zookeeper 事件通知机制
client端会对某个znode建立一个watcher事件，当该znode发生变化时，这些client会收到zk的通知，然后client可以根据znode变化来做出业务上的改变等。 比如某个锁(具有某种标识的节点)被释放了，其他客户端就可以被通知来获取锁。


# Zookeeper的广泛应用场景
1. 命名服务
2. 配置管理
3. 集群管理
4. 分布式锁
5. 队列管理


# 分布式系统中数据一致性和多处理器cache一致性对比


- [实例详解ZooKeeper ZAB协议、分布式锁与领导选举](http://dbaplus.cn/news-141-1875-1.html)
- [ZooKeeper 技术内幕：Leader 选举](http://ningg.top/zookeeper-lesson-2-leader-election/)
- [ZooKeeper part 2: building highly available applications, Ceilometer central agent unleashed](https://blogs.rdoproject.org/2015/08/zookeeper-part-2-building-highly-available-applications-ceilometer-central-agent-unleashed/)


# Paxos 论文
- [The Part Time Parliment (the original Paxos paper) by Leslie Lamport-最初版本带详细的证明过程和数学公式](http://research.microsoft.com/en-us/um/people/lamport/pubs/lamport-paxos.pdf)
- [Paxos Made Simple: another attempt at explaining Paxos by the original author by “deriving” the algorithm using the invariants of the consensus problem](http://research.microsoft.com/en-us/um/people/lamport/pubs/paxos-simple.pdf)
- [Paxos Made Live: an amazing paper from Google describing the challenges of their Paxos implementation in Chubby](http://static.googleusercontent.com/media/research.google.com/en//archive/paxos_made_live.pdf)
- [Fast Paxos-原作者改进](https://www.microsoft.com/en-us/research/publication/fast-paxos/)
- [Raft - An Understandable Consensus Algorithm. Raft is another conensus algorithm designed for humans to understand, which if you can tell from the above wall of text might be a problem with Paxos.](https://ramcloud.stanford.edu/raft.pdf)

## 知其所以然
 - [Paxos理论介绍(1): 朴素Paxos算法理论推导与证明](https://zhuanlan.zhihu.com/p/21438357)
 - [The Part Time Parliment (the original Paxos paper) by Leslie Lamport](http://research.microsoft.com/en-us/um/people/lamport/pubs/lamport-paxos.pdf)

# blog
 - [The Byzantine Generals Problem](https://marknelson.us/posts/2007/07/23/byzantine.html) 模拟拜占庭问题
 - [Understanding Paxos](https://understandingpaxos.wordpress.com/) 论文和实际之间的鸿沟
 - [Paxos By Example](https://angus.nyc/2012/paxos-by-example/)
 - [Understanding Consensus and Paxos in Distributed Systems](http://ifeanyi.co/posts/understanding-consensus/)
 - [分布式系列文章——Paxos算法原理与推导](http://linbingdong.com/2017/04/17/%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0%E2%80%94%E2%80%94Paxos%E7%AE%97%E6%B3%95%E5%8E%9F%E7%90%86%E4%B8%8E%E6%8E%A8%E5%AF%BC/)


## 工程
 - [Clustering by Consensus](http://aosabook.org/en/500L/clustering-by-consensus.html)
 - [Implementing Replicated Logs with Paxos](https://ramcloud.stanford.edu/~ongaro/userstudy/paxos.pdf)
 - [paxos made complex](http://paxos.systems/index.html)
 - [Visualization of a Paxos-based distributed state machine](http://rystsov.info/2016/05/01/paxos.html)


# unreliable
TCP不是可以保证可靠的通讯吗？为什么分布式系统中即使使用TCP也无法保证可靠呢？ 这里需要理解网络协议栈的分层和抽象与Socket编程的区别。我们在编写网络应用的时候，基本都是通过Socket编程接口来完成的，可以指定传输层使用TCP协议，但是TCP协议只是保证你一次调用发送的数据是可靠的，即保证该次数据不丢失，不重复且有序，TCP保障的是传输层的可靠性，但在我们应用层还是会出现阻塞和超时的，比如网络发生阻塞或者超时的时候(服务宕机，网络故障，这两种是无法区分的)，无论TCP有再好的机制，超时和Delay都还是会发生的，这种故障在Socket上的行为表现一般就是超时或者出错，不可达等，应用层一般会进行重试的处理！

那么如果在应用层面你对于某些请求是可以容忍重复执行，你就可以重试直到成功，但是如果你的应用不能容忍重复执行，那么你就不能重复发送或者需要保障机制来处理重复，这就是前者我们说的幂等操作，后者就是at-most-once操作，更具体的我们的应用可以分为可以忍受 at-least-once, at-most-once, exact-once. 这些都得在应用层次上由我们应用来保证，所以单靠TCP，应用程序也不一定是可靠的！但是TCP的一些算法和协议机制却可以在应用层利用。

一般如果信道不可靠，就会发生客户端不知道服务端到底是否收到请求，可以分一下情况:

 - 更本没有收到请求
 - 收到请求但是还未来得及执行
 - 收到了请求并已经提交执行

此时一般来说客户端选择重试机制！ 但是又会带来重复请求的情况(这种重复请求是善意行为，这和另外一种重复请求即某些产品的用户并发重复点击请求的这种场景有一定的区别)

## 重试机制
各种重试的自适应算法都有，简单的比如固定次数重试，指数衰减算法，碰撞检测算法等

## 如何判断重复的请求
### 协议
客户端和服务端协商，客户端的请求带版本号，新版本号必定是新请求，旧版本号是重复请求. 局限在于版本号的递增
```
client seq
   seq = get_and_increment(seq)
   for !ok {
     ok = rpc(seq)
   }

server seq
client_seq = clientmaps[client]
 if client_seq >= seq
     ok
 else 
     duplicated
```

### 只通过服务端来判断重复

## 网络分区
可以Ping通的一组服务器，突然由于网络原因，分成两组或者多组，组内之间可以通讯，但是组之间无法通讯。

### 脑裂
#### 主备系统
比如在主备系统中，假设当前状态是含有("k", "1") 的状态，同时客户端c1发送一个请求Get("k")给old primary 但是delay了， 由于partition，old primary 无法和view service 通讯,以至于view service 认为old primary 死亡，将 backup 设置为 new primary, 此时有一个客户端c2发送Put("k", "777")给 new primary， 然后c1的Get请求到达old primary，此时old primary还认为自己是primary，因为他一直无法和view service 通讯。然后客户端c1 会得到 "1" 而不是 "777".

此时各分区都认为有一个Primary，而Primary本来应该是唯一的。