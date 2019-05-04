---
title: "system-arch"
date: 2019-04-30 13:50
---

# 服务器体系结构
从系统架构来看，目前的商用服务器大体可以分为三类，即对称多处理器结构 (SMP ： Symmetric Multi-Processor) ，非一致存储访问结构 (NUMA ： Non-Uniform Memory Access) ，以及海量并行处理结构 (MPP ： Massive Parallel Processing) 。其中，SMP已经是过去式了，本科计算机组成原理都会以SMP来讲解体系结构，随着云时代的到来，服务器已经步入NUMA时代，知识需要更新一下。


## SMP
所谓对称多处理器结构，是指服务器中多个 CPU 对称工作，无主次或从属关系。各 CPU 共享相同的物理内存，每个 CPU 访问内存中的任何地址所需时间是相同的，因此 SMP 也被称为一致存储器访问结构 (UMA ： Uniform Memory Access) 。对 SMP 服务器进行扩展的方式包括增加内存、使用更快的 CPU 、增加 CPU 、扩充 I/O( 槽口数与总线数 ) 以及添加更多的外部设备 ( 通常是磁盘存储 ) 。
<img src="/static/images/CloudComputing/system-arch/smp.png" style="width:800px;height:300px;">
<caption><center><u> <font color="purple"> **SMP** </u></font> </center></caption>

SMP 服务器的主要特征是共享，系统中所有资源 (CPU 、内存、 I/O 等 ) 都是共享的。也正是由于这种特征，导致了 SMP 服务器的主要问题，那就是它的扩展能力非常有限。对于 SMP 服务器而言，每一个共享的环节都可能造成 SMP 服务器扩展时的瓶颈，而最受限制的则是内存。由于每个 CPU 必须通过相同的内存总线访问相同的内存资源，因此随着 CPU 数量的增加，内存访问冲突将迅速增加，最终会造成 CPU 资源的浪费，使 CPU 性能的有效性大大降低。实验证明， SMP 服务器 CPU 利用率最好的情况是 2 至 4 个 CPU 。

## MPP
多机器共同组成的一个大服务器，更加类似于MapReduce，实际上这种结构就是MapReduce。只不过起了个名字叫MPP

和 NUMA 不同， MPP 提供了另外一种进行系统扩展的方式，它由多个 SMP 服务器通过一定的节点互联网络进行连接，协同工作，完成相同的任务，从用户的角度来看是一个服务器系统。其基本特征是由多个 SMP 服务器 ( 每个 SMP 服务器称节点 ) 通过节点互联网络连接而成，每个节点只访问自己的本地资源 ( 内存、存储等 ) ，是一种完全无共享 (Share Nothing) 结构，因而扩展能力最好，理论上其扩展无限制，目前的技术可实现 512 个节点互联，数千个 CPU 。目前业界对节点互联网络暂无标准，如 NCR 的 Bynet ， IBM 的 SPSwitch ，它们都采用了不同的内部实现机制。但节点互联网仅供 MPP 服务器内部使用，对用户而言是透明的。

<img src="/static/images/CloudComputing/system-arch/mpp.png" style="width:800px;height:300px;">
<caption><center><u> <font color="purple"> **SMP** </u></font> </center></caption>
　　在 MPP 系统中，每个 SMP 节点也可以运行自己的操作系统、数据库等。但和 NUMA 不同的是，它不存在异地内存访问的问题。换言之，每个节点内的 CPU 不能访问另一个节点的内存。节点之间的信息交互是通过节点互联网络实现的，这个过程一般称为数据重分配 (Data Redistribution) 。

但是 MPP 服务器需要一种复杂的机制来调度和平衡各个节点的负载和并行处理过程。目前一些基于 MPP 技术的服务器往往通过系统级软件 ( 如数据库 ) 来屏蔽这种复杂性。举例来说， NCR 的 Teradata 就是基于 MPP 技术的一个关系数据库软件，基于此数据库来开发应用时，不管后台服务器由多少个节点组成，开发人员所面对的都是同一个数据库系统，而不需要考虑如何调度其中某几个节点的负载。

MPP (Massively Parallel Processing)，大规模并行处理系统，这样的系统是由许多松耦合的处理单元组成的，要注意的是这里指的是处理单元而不是处理器。每个单元内的CPU都有自己私有的资源，如总线，内存，硬盘等。在每个单元内都有操作系统和管理数据库的实例复本。这种结构最大的特点在于不共享资源
## NUMA
numa 是单个机器的体系结构，而mpp是多机器的。
<img src="/static/images/CloudComputing/system-arch/numa1.png" style="width:800px;height:300px;">
<caption><center><u> <font color="purple"> **SMP** </u></font> </center></caption>

### UMA
目前的PC可能还是UMA
<img src="/static/images/CloudComputing/system-arch/uma.jpg" style="width:800px;height:300px;">
<caption><center><u> <font color="purple"> **SMP** </u></font> </center></caption>


### UMA

目前很多服务器都是NUMA架构

<img src="/static/images/CloudComputing/system-arch/numa.jpg" style="width:800px;height:300px;">
<caption><center><u> <font color="purple"> **SMP** </u></font> </center></caption>

`numactl --hardware` 超过两个Node就是NUMA架构。也可通过 `ls /sys/devices/system/node/`
```
available: 2 nodes (0-1)
node 0 cpus: 0 1 2 3 4 5 6 7 8 9 10 11 24 25 26 27 28 29 30 31 32 33 34 35
node 0 size: 97901 MB
node 0 free: 41016 MB
node 1 cpus: 12 13 14 15 16 17 18 19 20 21 22 23 36 37 38 39 40 41 42 43 44 45 46 47
node 1 size: 98304 MB
node 1 free: 31863 MB
node distances:
node   0   1
  0:  10  21
  1:  21  10
```

`lscpu`，
```
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                48
On-line CPU(s) list:   0-47
Thread(s) per core:    2
Core(s) per socket:    12
Socket(s):             2
NUMA node(s):          2
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 79
Model name:            Intel(R) Xeon(R) CPU E5-2650 v4 @ 2.20GHz
Stepping:              1
CPU MHz:               2499.921
CPU max MHz:           2900.0000
CPU min MHz:           1200.0000
BogoMIPS:              4389.99
Virtualization:        VT-x
L1d cache:             32K              #数据缓存
L1i cache:             32K          #指令缓存 数据和指令分开缓存，属于冯诺依曼+哈佛体系结构
L2 cache:              256K
L3 cache:              30720K
NUMA node0 CPU(s):     0-11,24-35
NUMA node1 CPU(s):     12-23,36-47
```


- Socket就是主板上的CPU插槽;
- Core就是socket里独立的一组程序执行的硬件单元，比如寄存器，计算单元等;
- Thread：就是超线程hyperthread的概念，逻辑的执行单元，独立的执行上下文，但是共享core内的寄存器和计算单元
- 

NUMA体系结构中多了Node的概念，这个概念其实是用来解决core的分组的问题，具体参见下图来理解（图中的OS CPU可以理解thread，那么core就没有在图中画出），从图中可以看出每个Socket里有两个node，共有4个socket，每个socket 2个node，每个node中有8个thread，总共4（Socket）× 2（Node）× 8 （4core × 2 Thread） = 64个thread

<img src="/static/images/CloudComputing/system-arch/numa-node-group.png" style="width:800px;height:300px;">
<caption><center><u> <font color="purple"> **SMP** </u></font> </center></caption>

[numactl installation and examples](http://fibrevillage.com/sysadmin/534-numactl-installation-and-examples)

[Optimizing Applications for NUMA](https://software.intel.com/en-us/articles/optimizing-applications-for-numa)

[NUMA架构的CPU -- 你真的用好了么？](http://cenalulu.github.io/linux/numa/)

[Linux 的 NUMA 技术](https://www.ibm.com/developerworks/cn/linux/l-numa/index.html)

## k8s numa manager
[NUMA Manager Proposal](https://github.com/kubernetes/community/blob/4793277c981e7c7a5d9cbf1b2ab1003fc68384d3/contributors/design-proposals/node/numa-manager.md#new-component-numa-manager)
