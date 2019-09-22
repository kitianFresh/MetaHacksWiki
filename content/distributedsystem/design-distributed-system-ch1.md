---
title: "design-distributed-system-ch1"
date: 2019-07-15 22:54
---


07.15+累计第一天+《Design Distributed Systems by Brendan Burns》+第一章+心得:软件开发分两部分，高德纳的形式化算法编程+面向对象编程模式，分布式系统需要实现现代计算机程序所期望的可靠性，灵活性和可扩展性。 识别常见的模式和实践已经规范化并改进了算法开发和面向对象编程的实践。
将运行在同一台机器上的应用的各个组件拆分到不同的容器运行，可以达到资源分离，概念分离SoC，以及跨语言可重新性


07.16+累计第二天+《Design Distributed Systems by Brendan Burns》+第二章+心得:
模块化应用容器，单节点sidecar模式不仅仅可以以一种非侵入式（不用修改遗留应用的代码）方式改善增强存量应用的功能，还有可以重用组件和模块化；
举个例子，大多数引用都需要一个功能，想要查看debug你的应用中各个进程资源消耗，比如top命令的功能，你需要提供一种简单的方式让整个公司的所有应用都能够非常容易实现这个功能，你可能是提供一个特定语言实现的插件，让应用开发者能够通过你的插件包迅速拥有这种功能，但是你必须实现所有语言的插件包来满足不同语言的应用；更简单有效的方法，是通过sidecar的模式部署一个sidecar容器，和开发者的应用部署在同一个namespace下，共享进程命名空间，这样sidecar容器通过top就可以拿到所有的进程资源消耗了。
当然也是一种权衡trade-offs, 如果你对性能要特别看重，可能就必须得编写代码插入你的应用了，sidecar模式只是一种通用的解决方式，对于极少数特殊需求，可能还是需要特殊对待

sidecar模式用于构建PaaS

边车模式应用原则
1. Parameterizing your containers（容器启动参数，环境变量或者通过命令行脚本）
2. Creating the API surface of your container（容器需要一个API平面）
3. Documenting the operation of your container（文档化）


07.17+累计第三天+《Design Distributed Systems by Brendan Burns》+第三章+心得:
单节点ambassador模式，本质上就是引入了服务发现和代理机制，这部分逻辑不用插入到应用客户端代码频繁更新迭代，而是通过ambassador容器维护运行
1. ambassador容器实现sharding，很多时候我们的存储需要进行shard操作，由原来的单一节点，sharding分片到多个节点，导致我们的应用容器代码需要变更，原来连接一个节点，现在需要客户端连接多个节点，一种是将shard代码逻辑以客户端的形式插入应用中，应用部署会变的复杂，但是存储服务不变；另一种是将shard代码逻辑构建到存储服务中，此时存储服务还需要提供一个无状态load balancer将流量打到存储服务，存储服务部署变得复杂。一个例子就是 sharding redis 服务，开始你只是用了一个节点的redis，后来需要增加到三个节点，此时可以通过一个 ambassador 分流容器，原来的应用不用更改，ambassador容器会代理请求，然后sharding到不同的redis
2. ambassador容器实现服务发现代理service broker/discovery
3. ambassador 容器实现服务AB测试分流，AI模型往往进行AB测试分流，通过一个分流容器，很容易将请求分流到prod 和 test后端服务


07.18+累计第四天+《Design Distributed Systems by Brendan Burns》+第四章+心得:
单节点adapter模式，通过适配器转换格式，兼容各种不同的格式和实现，并转换到统一的格式标准
1. 日志转换
2. 监控数据收集
3. 健康检查容器


07.19+累计第五天+《Design Distributed Systems by Brendan Burns》+第五章（第二部分ServingPatterns）+心得:
## Replicated Load-Balanced Services
可水平扩展的无状态服务，每个服务的角色一样，都可以分摊流量，节点可以随意增减，这种服务必须要加一个load balancer 在前面分流（nginx，美团是oceanus）
### 1. Stateless Services,
无状态系统实现副本化，通常是通过冗余节点和水平扩展，不管服务多么微小，你至少需要2个副本实现高可用，例如实现 三个9 (99.9%), 意味着每天服务不可用必须在1.4分钟以内（24x60x0.001）,这还不算软件升级更新，如果你的服务是每小时都需要持续集成CI/CD, 那么99.9%的高可用就意味着不可用时间压缩到3.6秒/小时(1x60x60x0.001)；通常而言通过 load balancer 负载均衡就可以实现无状态容器服务的replicate；

probe readiness，服务ready和服务启动是两码事，服务需要一个url来确认是否是ready状态（美团OCTO），以便于负载均衡做选择。一般在kubernetes中Pod支持自定义检查ready的脚本。

session tacked services，有的replicated模式是把请求任意达到某个节点，而有的不是，必须要根据请求算出指定的节点，这种往往就是节点有用户缓存，为了提高缓存命中率，必须根据不同请求分发到不同节点，这种往往在load balancer中通过一致性hash解决可扩展性问题; 这里的tacking都是在网络层，source ip + destination ip 的hash的方式转发，只能在集群内网（源ip和目标ip）进行，如果是外网，则需要 应用层 的tacking，比如使用 cookie

### 2. Application-Layer Replicated Services；
1 中都是在网络层(IP)进行的控制，而在应用层(HTTP)，也是需要缓存，此时往往通过web proxy代理的形式


## Caching Layer
无状态服务和load balancer之间，可能还会引入缓存层


07.20-21+累计第七天+《Design Distributed Systems by Brendan Burns》+第六章(Sharded Services)+心得:
## Sharded Services 
分片服务和副本服务（Replicated Service）不同的地方在于，副本服务中每个副本是对等的，任意一个请求可以打到任意一个副本，但是分片服务不是，通常一个请求只能打到特定的分片节点。replicated service 通常用来构建stateless 服务，而 sharded service通常用来构建stateful服务；
### sharded caching
衡量cache的性能重要指标除了高可用稳定性性能，就是缓存命中率（hit rate）,不命中意味着缓存层的出现反而是在降低系统的性能，缓存不命中意味着请求处理需要都走弯路。

分片数据的目的就是服务的状态数据太大以至于一个机器放不下；缓存分片服务中分片函数的设计非常重要，使用最多的是一致性hash函数。

分片服务比仅仅用于缓存服务，他主要就是数据在一台服务器装不下而进行分片，比如游戏分服务区，当游戏选手很多的时候，一般会根据地理位置作为key进行hash分片存储到不同的服务区，因为地理位置上相隔很远的选手一般很少有高频联系。

### hot sharding system
有时候会出现即使分片函数设计的再好，但是总会有突发请求全部打到同一个分片上，导致某个分片过载（热分片）。

因此，热分片系统 这里再在sharded service 中引入 replicated service，也就是分片本身也可以是一个sharded service，比如本来有A/B/C 三个分片分别位于三台机器，当请求变大的时候，A分片过载，此时可以将A分片进行replicate，这样分片A就会变成两个replicated service分布到两台机器，B和C收缩至同一个机器。为什么B/C要收缩到同一台机器？我想应该是为了保证资源利用率的均衡，B和C分片此时相对来说不热，因此放到同一台机器足够了。


07.22+累计第八天+《Design Distributed Systems by Brendan Burns》+第七章(Scatter/Gather)+心得:
## Scatter/Gather
前面几章中，stateless replicated 模式是从rps(requests processed per second)服务器的吞吐量角度进行扩展， sharded data 模式是从数据大小的角度进行扩展，而Scatter/Gather 模式，则是从一个请求的处理时间的角度进行扩展；

Scatter/Gather 就是shard computation的一种模式，他是对计算进行分发（当然数据shard也包含在其中），map/reduce 就是其中一个例子。
### Scatter/Gather with Root Distribution
分布式文档搜索, 一个请求会被同时发到每个叶子节点，叶子节点都会并行计算处理，结果返回到根节点后由根节点合并。

### Scatter/Gather with Leaf Sharding
在上述root distribution中，由于叶子节点都是单个的，任意一个叶子节点出故障等原因无法及时返回结果，这次请求就会失败，

### straggler problem
选择正确的叶子数非常关键，因为scatter/gather 模式的总返回时间其实取决于最慢的那个叶子节点的返回时间；比如每个叶子replica处理请求的TP99 是2s（意思是由99%的请求小于2s，剩下有1%的请求一定大于2s），那么一个请求有1%的概率是大于2s，看起来还不错，这意味着100个用户中只有1个用户体验比较差，但是如果你的replica比较多，比如是5，那么就会是5%的概率大于2s（0.99x0.99x0.99x0.99x0.99==0.95）,这样以前的TP99 变成了TP95，服务的SLA其实降低到毁灭性的地步，是想如果你的leaf数目等于100，那么这个数据只会更低，变成64%的概率服务时间大于2s

增加叶子节点并不总是能够加速，第一因为每个叶子节点处理都是有开销的(尽管比较小，但是叶子多了以后开销就累计多了)，第二因为straggler problem，短板效应

相比于其他模式，TP99 在scatter/gather 系统中非常重要，因为每个用户请求其实变成了多个请求拆分处理

### scaling scatter/gather
其实，和replicated sharded service一样，我们可以对每个叶子节点进行replicated操作，这样，每个叶子都实现副本化，这样就不会发送叶子节点故障而无法响应的情况，通过多副本叶子节点，你可以很轻松的更新升级部署叶子节点还不用影响线上服务。当然，代价就是需要更多的机器。


07.23+累计第9天+《Design Distributed Systems by Brendan Burns》+第八章（Functions & Event-Driven）+心得:
FaaS（Function-as-a-Service） 是目前最先进的云服务，但是FaaS服务有很多挑战和缺点。

### 好处
FaaS 简化了从代码到服务部署的过程，对于开发者来说，连服务器的概念都没有了。同时，一个Function（无状态的，FaaS只用于无状态函数）可以随时增加实例来弹性应对高峰流量。

### 挑战
1. 基于FaaS的系统无法再获取应用的完整的代码视图，因为函数都是部署在其他地方，调试和跟踪问题异常艰难。还会出现函数循环依赖的现象，以至于应用一直循环递归调用。FaaS需要完善的调试和监控工具，这是挑战之一。
2. 对于需要后台异步处理的服务请求，FaaS必须运行在支持这种可以在后台开启处理进程的环境中，比如编解码视频，压缩日志文件，Function对于这些请求的响应一般是异步的，后台处理，你需要启动一个后台进程长时间运行起来，这样你还是得使用容器或者虚拟机(pay-per-consumption) 而不是只有FaaS(pay-per-request)
3. 需要加载数据到内存并处理的服务，不适合改造成FaaS，这样会造成更大的时间消耗，因为FaaS本来就是调用的时候才启动，这里就是冷启动，会加载数据，这样很慢
4. 实际上，pay-per-request（按每次请求付费） 模型初期看起来比 pay-per-consumption（按虚拟机或容器运行消耗收费） 便宜，按请求付费而不是按照计算消耗付费，但是随着请求量剧增，由于每次请求都需要付费，此时还不如长时间运行的容器或者虚拟机省钱。

总体而言，我认为FaaS炒作大于他实际的用途，缺点还是很多的，只是对于公有云厂商来说，FaaS可以通过长尾效应，靠走量获得一定利润，但是FaaS只会是很小的市场，所以只能公有云厂商这种体量巨大的提供商做FaaS才会赚钱，对于私有云，小企业，FaaS其实只会占很小一部分

## pattern for FaaS
### decorator pattern
装饰器模式，类似传统编程中的装饰器，不过这里采用FaaS实现，kubeless是一个基于kubernetes的faas框架，可以很容易实现faas，你只需要写好你的函数代码文件，然后通过kubeless创建就可以了。
### two-factor authentication
密码+手机就是双因子认证，他更加安全，同时知道密码和拿到手机才能登陆系统。



07.24-26+累计第12天+《Design Distributed Systems by Brendan Burns》+第八章（Ownership/Leader Election）+心得:

前面的章节描述的模式分别是关于扩展QPS，扩展内存状态数据，扩展处理时间，本部分最后一章节，是关于扩展任务分配。
## 是否需要选主系统
分布式选主通常是设计高可用分布式系统中最复杂和最重要的部分。如果采用单点模式，在容器编排系统如kubernetes中，容器crash或者hang，重启即可，大概花费2s，假设容器一天崩溃一次，那么可用性大概是4个9（1-2/86400=99.997%）;如果是容器所在宿主机宕机，kubernetes会自动迁移容器到其他机器，大概花5分钟（原因是宕机感知一般设置5分钟网络不可达等，感知宕机是比较难的问题），如果集群中每天有一个宿主机宕机（假设服务器宕机率万分之一，那么拥有10000台节点的集群，每天都会发生宕机），此时只能做到2个9

如果你的SLA要求比较高比如4个9，那么必须要引入更多replicas，并且只有其中一个replica是owner，对外提供服务，也就是常说的active/standby 模式，同一时间只会有一个replica是master对外提供服务，其他是备用节点

选主主要涉及两个问题，如何选master以及master挂了以后如何选出新的master；有两种方式实现选主，第一种是实现分布式一致性协议如Paxos或者RAFT，但是比较复杂。另外是直接使用已经帮助你实现了一致性算法的分布式 key-value 存储组件比如Zookeeper/Consul/Etcd. 这些系统都会提供对一个key的类似compareAndSwap 的原子操作原语。

## 选主实现
CAS 的原理都是一样的，在单机系统是由硬件指令帮助完成，而在分布式系统则是由key-value store负责这个原子操作，比较和交互一定是原子的
```Go
var lock = sync.Mutex{}
var store = map[string]string{}
func compareAndSwap(key, nextValue, currentValue string) (bool, error) {
    lock.Lock()
    defer lock.Unlock()
    if _, found := store[key]; found {
        if len(currentValue) == 0 {
        store[key] = nextValue
        return true, nil
        }
        return false, fmt.Errorf("Expected value %s for key %s, but found empty", currentValue, key)
    }
    if store[key] == currentValue {
        store[key] = nextValue
        return true, nil
    }
    return false, nil
}
```
对于分布式的CAS，都必须要提供另一个参数，ttl（time-to-live），也就是timeout，因为分布式通讯的假设就是网络不可靠，这个CAS操作虽然能够在成功的情况下保证原子，但是如果网络中断，那么key就会置为空。

### 分布式锁
基于CAS原理
```Go
func (Lock l) simpleLock() boolean {
    // compare and swap "1" for "0"
    locked, _ = compareAndSwap(l.lockName, "1", "0")
    return locked
}
```
但是锁可能不存在，比如首次索要，需要判断具体的两种情况。
```Go
func (Lock l) simpleLock() boolean {
    // compare and swap "1" for "0"
    locked, error = compareAndSwap(l.lockName, "1", "0")
    // lock doesn't exist, try to write "1" with a previous value of
    // non-existent
    if error != nil {
        locked, _ = compareAndSwap(l.lockName, "1", nil)
    }
    return locked
}
```
传统的互斥锁实现是阻塞式的，获取失败就sleep一会儿，等待下一次获取
```Go
func (Lock l) lock() {
    while (!l.simpleLock()) {
        sleep(2)
    }
}

func (Lock l) unlock() {
    compareAndSwap(l.lockName, "0", "1")
}
```
上述锁的实现问题就是每次失败都需要睡眠阻塞，这种polling形式性能很差. 还好，key-value store 都提供通知机制.
```Go
func (Lock l) lock() {
    while (!l.simpleLock) {
        waitForChanges(l.lockName)
    }
}
func (Lock l) unlock() {
    compareAndSwap(l.lockName, "0", "1")
}
```
最后进程可能会挂，网络会中断，CAS是走网络的，因此需要设置ttl
```Go
func (Lock l) simpleLock() boolean {
    // compare and swap "1" for "0"
    locked, error = compareAndSwap(l.lockName, "1", "0", l.ttl)
    // lock doesn't exist, try to write "1" with a previous value of
    // non-existent
    if error != nil {
        locked, _ = compareAndSwap(l.lockName, "1", nil, l.ttl)
    }
    return locked
}
```
> 特别注意，使用分布式锁的时候，保证任何进程都不能持有锁超过TTL时间。最佳实践是设置一个watchdog timer 当你获取锁的时候，watchdog 包含断言可以crash你的程序，如果在你调用unlock之前TTL就expired; 详细可以查看美团云的分布式锁冲突和超时检查机制

但是以上的版本还是有bug，例如在一下场景，P2 和 P3 同时获取锁
1. Process-1 obtains the lock with TTL t.
2. Process-1 runs really slowly for some reason, for longer than t.
3. The lock expires.
4. Process-2 acquires the lock, since Process-1 has lost it due to TTL.
5. Process-1 finishes and calls unlock.
6. Process-3 acquires the lock.

但是key-value 又提供了 resource version 为每一个写操作。 这样，P1 由于过期，key被重置，resource version发生变化，此时P1 调用 unlock是无法写的。这样P1就不会影响P2/P3

```Go
 func (Lock l) simpleLock() boolean {
    // compare and swap "1" for "0"
    locked, l.version, error = compareAndSwap(l.lockName, "1", "0", l.ttl)
    // lock doesn't exist, try to write "1" with a previous value of
    // non-existent
    if error != null {
        locked, l.version, _ = compareAndSwap(l.lockName, "1", null, l.ttl)
    }
    return locked
}

func (Lock l) unlock() {
    compareAndSwap(l.lockName, "0", "1", l.version)
}
```

这里说明一下，以上分布式锁的实现只是比较初级的形式，分布式锁其实还有更高级的实现形式，zookeeper中的watch机制其实有惊群效应，如果一把锁竞争的过于激烈，那么该锁就会有上千上百的客户端在监听，每次zookeeper/etcd 完成CAS操作通知的时候需要通知很多客户端，导致网络开销和性能问题，因此现在设计一般都是排队锁，每次通知只会通知到比自己大一号的客户端。

### 实现所有权Ownership

锁帮助建立短暂的ownership是有效的，但有些场合其实是需要获取一段时间的的所有权。比如在高可用的kubernetes master组件中，kube-scheduler 有多个replicas，但是同一时刻只有一个active的，并且一旦一个replica成为active，他会一直保持active直到进程因为某些原因挂掉。

那其中一种方法是把TTL加长到一周，这样获取锁的replica就会active一周，这样就实现了长期ownership，但是一旦replica在TTL时间内挂掉，其他replica需要等待一周到才有机会获取ownership，这就无法提供服务了，肯定不行。

我们可以提供一种可重建锁（renewable lock），自己可以重新new自己（有点类似 reentrant lock）
```Go
func (Lock l) renew() boolean {
    locked, _ = compareAndSwap(l.lockName, "1", "1", l.version, ttl)
    return locked
}
```
在一个单独线程重复获取锁的过程，自己重复获取锁，一旦无法获取，就做handleLockLost 的逻辑
```Go
for {
    if !l.renew() {
        handleLockLost()
    }
    sleep(ttl/2)
}
```

### 在etcd中实现租用(leases)


### 处理并发修改（作者的图太突兀，而且他自己也没说清楚，此部分略过，后续再详细查阅资料）
上述的锁机制尽管已经很好了，但是还是有可能发生两个replicas同时认为自己持有锁。比如在过度调度的机器上，处理器容易出现卡顿，锁超时，此时handleLockLost()函数会很快执行，但是会有一段时间，replica还认为它自己持有锁。

一般通过检查锁是否保留，此函数在任何临界区执行，那两个replica同时获取ownership的概率会大大降低。然而，这也不能完全消除。但是还有最终办法是引入version和主机名字。


# 第三部分（批处理模式）
第一二部分都是在线服务，长时间运行，第三部分主要是批处理，短时间运行的服务模式。

## Work Queue System
work queue 是最简单的批处理模型，在work queue系统中，往往是一批work需要执行。每一个work piece彼此完全独立，能够被单独处理，没有相互依赖关系。通常来说，work queue 系统的目标就是保证每一个 work piece 能够在规定时间内完成，Workers 能够扩容（scale up） 或 缩容（scale down）来保证一个work能够被及时处理。

## Generic Work Queue System
work queue 是一种诠释分布式系统模式好处的理想方式。work queue的多数逻辑是独立于 工作执行者和工作交付者的。（简单点，就是消费者和生产者）

因此一个基于container 的work queue系统其实也即是三部分，两个接口：work item 生产者接口和work item 消费者接口，一个共享的work queue（严格的说应该是 work queue manager）

### Source Container Interface
可以使用 ambassador 模式实现 application-specific 的 source container 来和 work queue manager 协作，比如source 可以是列出图片列表，文件列表，或者是一个存在于像 Kafka 或 Redis 这样的发布订阅系统中的queue等等，work queue manager 负责拉取这些source中的work item.

### Worker Container Interface
同样在 work queue manager 和 worker 之间也需要一个规范的接口，但是和上述的source interface有些不同之处。

第一，它是一种 one-off API, 接口只调用一次，来启动work，并且在worker容器的运行周期内，再无其他API调用。第二，worker container不和 worke queue manager处于同一个CGroup，worker container是通过编排API（如kubernetes api）启动，并被调度到它自己的CGroup中。

这也意味着work queue manager需要通过远程调用来启动worker container，同时就意味着我们需要注意安全性以防止恶意用户注入额外的工作到集群中。具体而言，对于 worker container 我们可以使用基于文件的API，也就是说，每当worker container被创建的时候，它会接受一个key=WORK_ITEM_FILE 的环境变量，这个文件指向容器的本地文件系统，数据字段就写在这个文件上。在kubernetes中我们可以很容易通过 ConfigMap 来实现，因为ConfigMap 可以作为文件挂载到容器组中(Pod)。

### Shared Work Queue Infrastructure
剩下的就是实现可重用的work queue了。kubernetes提供了Job这个对象，能够保证work queue的可靠执行。Job可以配置worker container是一次执行还是可重复执行知道成功。Job还有Annotation可以帮助我们标记每一个Job对应处理的work item，从而可以知道哪些工作失败或者成功。

> Repeat forever
>Get the list of work items from the work source container interface.
>Get the list of all jobs that have been created to service this work queue.
>Difference these lists to find the set of work items that haven’t been processed.
>For these unprocessed items, create new Job objects that spawn the appropriate worker container.

```Python
import requests
import json
from kubernetes import client, config
import time
namespace = "default"
def make_container(item, obj):
    container = client.V1Container()
    container.image = "my/worker-image"
    container.name = "worker"
    return container
def make_job(item):
    response = requests.get("http://localhost:8000/items/{}".format(item))
    obj = json.loads(response.text)
    job = client.V1Job()
    job.metadata = client.V1ObjectMeta()
    job.metadata.name = item
        job.spec = client.V1JobSpec()
    job.spec.template = client.V1PodTemplate()
    job.spec.template.spec = client.V1PodTemplateSpec()
    job.spec.template.spec.restart_policy = "Never"
    job.spec.template.spec.containers = [
        make_container(item, obj)
    ]
    return job
def update_queue(batch):
    response = requests.get("http://localhost:8000/items")
    obj = json.loads(response.text)
    items = obj['items']
    ret = batch.list_namespaced_job(namespace, watch=False)
    for item in items:
        found = False
        for i in ret.items:
            if i.metadata.name == item:
                found = True
        if not found:
            # This function creates the job object, omitted for
            # brevity
            job = make_job(item)
            batch.create_namespaced_job(namespace, job)
config.load_kube_config()
batch = client.BatchV1Api()
while True:
    update_queue(batch)
    time.sleep(10)
```

#### Hands On: Implementing a Video Thumbnailer

### Dynamic Scaling of the Workers
work queue 不管在单机上还是分布式，都有一个重要功能需要支持，就是动态调整worker数目，来平衡资源使用效率和任务完成时间。根据生产者和消费者的速度，监控队列的各项数据指标，从而对worker数目进行扩缩容。

### The Multi-Wroker Pattern
本书的主题之一就是学会使用容器来封装和重用代码。work queue系统中也同样适用。比如我们可以利用许多不同种类单一职责的worker container，实现一个聚合worker从而完成更复杂和高级的功能，而且还能实现更细的组件重用。比如原来是做人脸识别和标记的系统，现在改成做猫脸识别和标记系统。完全可以把识别container和标记container，模糊container单独实现，而后通过一个 Multi-worker Aggregate container 来分别适应人脸或者猫等等。

07.29+累计第15天+《Design Distributed Systems by Brendan Burns》+第11章（Event-Driven Batch Processing）+心得:
# Event-Driven Batch Processing
上一章讲述的是通用的work queue处理框架，但是仅仅针对一个输入一个输出的单转换处理。还有一些批处理系统，需要执行不止一次单个动作，你可能需要从一个输入同时产生多个输出，需要将不同的工作队列连接起来，从而实现一个queue的输出成为另一个queue的输入，而且上一个步骤中完成需要通过事件通知到下一个步骤中的work queue， 这种事件驱动的批处理系统，通常也称作 工作流（workflow） 系统。

## Copier
Copier 模式将单个work items 工作流分发到多个一样的输出流中。一个例子就是 视频渲染，当渲染视频的时候可能需要不同的格式，比如 4-KB 的高清方案，1080p 的数字流，为移动和网速慢的低分辨率的格式，或者一个GIF缩略图，所有的输入都是同一个视频，但是处理是不同的work queue.

## Filter
Filter 模式很简单，就是过滤出某些符合条件的work items. 比如在用户注册服务中，某些用户勾选了愿意通过邮件接受促销和其他信息，有些没有，你就可以使用filter。通常情况下，你可以使用ambassador模式包装一个work queue source，source container 提供完整的work items列表，而filter container 则基于过滤规则调整items.

## Splitter
splitter 模式是将一个输入拆分成两部分，分别放入两个单独的work queue。他是基于拆分规则，对work item进行拆分。一个例子就是线上订单系统，用户需要通过邮件或者短信接收购物提醒。实际上，一个splitter 可以是copier，你会发现，copier其实也可以实现这种功能，只是需要在work queue做一下处理。

## Sharder
Sharder 是 splitter 的一个更加通用的形式。sharder 可以把单个queue划分成奇偶两个集合。使用 sharder 的原因有几个，第一是可靠性，队列被shard以后，单个workflow的失败不影响整个系统。

## Merger
和Copier相反，Merger就是将多个输入work queue转换到单个work queue. 比如，你有许多不同的代码仓库同时添加了新的commits，你想要使用这些commits中的每一个做 build-and-test. 如果为每个代码仓库建立单独的构建系统，将非常难以扩展。你可以将这个建模成多个不同的work queue source,他们各自提供自己的commits, 然后通过merger adapter将这些commits合并成一个单独的stream去构建。

## Hands On: New User Sign-Up Event-Driven WorkFlow
新用户注册，一般会有两个阶段，第一个阶段是用户认证，第二个阶段是发送确认email，让用户选择email或者短信或者两种来进行提醒。

## Publisher/Subscriber Infrastructure
Kafka 是比较容易实现的消息队列，提供发布订阅模式。


07.31+累计第18天+《Design Distributed Systems by Brendan Burns》+第12章（Coordinated Batch Processing）+心得:
# Coordinated Batch Processing
前面的章节，我们主要讲解的是那些拆分工作后分发到多个节点，并行计算。但是，有时候在处理workflow的时候，需要等待前一个阶段的整个数据集都处理完成以后，才能进入workflow的下一个阶段。
处理这种需要协调的workflow，可以选择直接用前面讲到的work queue，把所有的queue的结果merge一下，但是merge只能简单的把两个queue混合进一个queue做额外的处理，无法保证在处理以前整个数据集已经准备好。
## Join(or Barrier Synchronization)
在批处理模式下，我们需要一种更强的协调原语，他就是 join pattern. Join 在单机就类似 join thread，主线程等待从线程完成，可以通过从线程在主线程中调用join方法。并发编程中就是所谓的 Barrier Synchronization. Java 里面有java.util.concurrent.CyclicBarrier

## Reduce
和Join相比，Reduce有些类似，但是 reduce 不用等待所有的数据被处理完才进行，这意味着reduce是可以和Map/Shard 并行的。



