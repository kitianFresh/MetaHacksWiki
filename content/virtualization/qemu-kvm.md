---
title: "qemu-kvm"
date: 2018-03-27 17:10
---

# qemu-kvm
[KVM: Kernel-Based Virtual Machine](https://www.linux-kvm.org/page/HOWTO) 是基于内核的虚拟机，目前已经属于Linux内核的一个可加载模块，通过调用Linux内核本身的功能，实现对CPU和内存的虚拟化。本质上，KVM是管理虚拟硬件设备的驱动，该驱动使用字符设备/dev/kvm（由KVM本身创建）作为管理接口，主要负责vCPU的创建，虚拟内存的分配，vCPU寄存器的读写以及vCPU的运行。
[Qemu](https://www.qemu.org/)　就是一个计算机硬件模拟器，可以虚拟出一台硬件设备齐全的机器。QEMU有两种工作模式：系统模式，可以模拟出整个电脑系统，另一种是用户模式，可以运行不同与当前硬件平台的其他平台上的程序（比如在x86平台上运行跑在ARM平台上的程序）
在0.9.1及之前版本还可以使用kqemu加速器（可以理解为QEMU的一个插件，用来提高QEMU的翻译性能，支持Windows平台），但1.0以后版本就只能使用qemu-kvm（只支持Linux）进行加速了，1.3版本后QEMU和QEMU-KVM合二为一了。

[QEMU-KVM](https://huangwei.me/wiki/tech_cloud_kvm_qemu_libvirt_openstack.html) 由于kvm只能虚拟CPU和内存，并且完全处于内核态，因此用户无法直接控制。KVM运行在内核空间，QEMU运行在用户空间，实际模拟创建，管理各种虚拟硬件，QEMU将KVM整合了进来，通过/ioctl 调用 /dev/kvm，从而将CPU指令的部分交给内核模块来做，KVM实现了CPU和内存的虚拟化，但kvm不能虚拟其他硬件设备，因此qemu还有模拟IO设备（磁盘，网卡，显卡等）的作用，KVM加上QEMU后就是完整意义上的服务器虚拟化。

另外，由于qemu模拟io设备效率不高的原因，现在常常采用半虚拟化的virtio方式来虚拟IO设备。综上所述，QEMU-KVM具有两大作用：

 1. 提供对cpu，内存（KVM负责），IO设备（QEMU负责）的虚拟
 2. 对各种虚拟设备的创建，调用进行管理（QEMU)负责

很不幸的是，qemu　kvm 只支持　linux 平台，Mac 上无法使用 enable-kvm; 
## 参考
 - [KVM-Qemu-Libvirt三者之间的关系](http://blog.51cto.com/changfei/1672147)

# qemu启动虚拟机
启动虚拟机之前，你必须得有一块磁盘给虚拟机使用，就像我们有了一台裸机之后，需要安装系统到磁盘上或者直接从已经安装过操作系统的磁盘/光驱/U盘启动一样。因此，这里有两种方式:
## 从ISO镜像启动(类似于物理机安装操作系统)
你也可以创建一块虚拟裸盘（我们一般称之为镜像、磁盘镜像），然后选择从ISO光驱等启动虚拟机，然后你会和使用物理裸机一样开始进入安装操作系统的正常步骤，安装完成之后，你的虚拟裸盘就变成了真正的带操作系统的系统盘了。

首先创建一个空盘
```
qemu-img create -f qcow2 ubuntu-server.qcow2 20G
```
从ISO光驱启动, 这里的boot其实类似于我们物理机设置BIOS里面的启动项的启动顺序，比如U盘,磁盘，硬盘，光驱启动。d指的是从光驱启动安装ubuntu-16.04.3-server-amd64.iso，-c从硬盘启动，-n是从网络启动.
一般情况是，使用cdn这个顺序，就是直接从硬盘启动，如果硬盘没有安葬操作系统，就从光驱启动，进行操作系统安装了。最后是从网络。
```
qemu-system-x86_64 -m 2048 -smp 4 -hda ubuntu-server.qcow2 -cdrom /home/kinny/Tools/ubuntu-16.04.3-server-amd64.iso -enable-kvm -boot cdn
```
参数说明:
```
-m 2048: 虚拟机内存是2048MB 
-smp 4: 虚拟机有4个vcpu  
-hda: 指定了硬盘是那个虚拟磁盘，这里用刚刚创建的ubuntu-server.qcow2；
-cdrom: 指定启动的cdrom，可以用iso文件，也可以用机器的光驱，我们选择用iso文件;如果用光驱尝试-cdrom /dev/cdrom；
-boot: 指定启动的时候从磁盘，硬盘，光驱启动，我们安装的时候选择从光盘启动，所以用d；
```
关于以上的参数选项说明，可以参考详细的[wiki](https://wiki.gentoo.org/wiki/QEMU/Options). 另外，安装 ubuntu server 版本的时候，如果你选择中文版，会出问题，`virtualbox安装ubuntu-16.04.1-server-amd64出现“无法安装busybox-initramfs”错误。向目标系统中安装busybox-initramfs软件包时出现一个错误。请检查/var/log/syslog或查看第四虚拟控制台以获得详细信息。`这貌似是 ubuntu 的bug,换成安装英文版就可以成功了！反正从采用虚拟机安装中文版都会出现这种错误，不管是qemu/kvm 还是virtualbox vmware.

## 从已经含有操作系统的磁盘镜像启动（类似于从已经安装过操作系统的物理机启动机器）
你可以直接拿到一个已经安装过操作系统的虚拟磁盘，从这个虚拟磁盘启动虚拟机，
```
qemu-system-x86_64 -m 2048 -smp 4 -hda ubuntu-server.qcow2 -enable-kvm
```

# qemu 网络
qemu 网络主要由两部分构成
 - 虚拟机(guest)内部的虚拟网卡设备(PCI network card)
 - 和虚拟网卡交互的网络后端，即网络类型，用户网络(user networking)，Tap 网络，VDE网络，Socket网络
## User mode networking
用户模式网络是一种qemu虚拟机比较简单的网络配置方法，这种网络只允许虚拟机访问宿主机网络以及外网，但不允许其他虚拟机或者宿主机以及局域网和互联网上的机器访问该虚拟机，就是只能出去，无法进来。因此他也不支持 ICMP 协议，如ping等。只支持 TCP/UDP协议。这种网络模式非常类似于virtualbox　或者 VMware 的NAT模式。
```
qemu-system-x86_64 -m 2048 -hda ubuntubase.qcow2 -smp 4 -netdev user,id=network-ubuntubase -device e1000,netdev=network-ubuntubase,mac=52:54:e4:99:78:31
```
执行完以上命令后，你在宿主机上 `ip a` 或者 `ifconfig` 是看不到该虚拟机的网卡设备的，这种用户模式网络对外不可见(除了虚拟机本身).　默认不给出网络设置的话，qemu默认创建出如下如所示的用户模式网络.

<img src="/static/images/Virtualization/qemu-kvm/qemu-default-user-network(slirp).png" style="width:500px;height:300px;">
<caption><center><u> <font color="purple"> **slirp** </u></font> </center></caption>


如果想要从宿主机上通过scp拷贝文件到虚拟机,则可以通过端口转发，`-device e1000,netdev=network-ubuntubase,mac=52:54:e4:99:78:31,hostfwd=tcp::5555-:22`. 这样你就可以通过 `scp -P 5555 file.txt tq@localhost:/` 往虚拟机传文件了。但是记住前提是要把宿主机的公钥拷贝到虚拟机ssh目录下的authorized_keys文件里，且虚拟机需要安装并运行sshd,即openssh-server. `sudo apt-get install openssh-server`
如果hostfwd 属性不认识，端口重定向还可以采用如下的形式,直接将宿主机的端口重定向到虚拟机的端口.
```
qemu -m 256 -hda disk.img -redir tcp:5555::80 -redir tcp:5556::445 &
...
mkdir -p /mnt/qemu
mount -t cifs //localhost/someshare /mnt/qemu -o user=test,pass=test,dom=workgroup,port=5556
firefox http://localhost:5555/
```

还可以设置网络地址范围改变上图的默认值. `-netdev user,id=network-ubuntubase,net=192.168.76.0/24,dhcpstart=192.168.76.9`
用户模式网络一般只会用于简单的使用虚拟机，而且他的性能很差，没有Tap性能好.
 - there is a lot of overhead so the performance is poor
 - in general, ICMP traffic does not work (so you cannot use ping within a guest)
 - on Linux hosts, ping does work from within the guest, but it needs initial setup by root (once per host) -- see the steps below
 - the guest is not directly accessible from the host or the external network

## Tap networking
[TUN/TAP](https://en.wikipedia.org/wiki/TUN/TAP)都是虚拟网络内核设备。其实就是内核虚拟出来的网卡。TUN 隧道网络设备，是工作在第三层的网络设备，即他直接操作的是IP报文。TAP 是工作在数据链路层的网络设备，他直接操作MAC帧。TUN一般用来做路由器，而TAP一般用来创建网桥。
操作系统可以将数据包通过TUN/TAP转发给附着在他们上面的用户态应用程序，反之，用户态应用程序也可以将数据包通过TUN/TAP转发给操系统网络协议栈处理。这种网络模式非常灵活，可以实现virtualbox,VMware中的桥接模式，使得虚拟机网卡和宿主机网卡处于同等地位，就好像一个局域网上另一台真实物理机的网卡一样。

### 桥接模式
Tap 设备是支持Linux bridge driver即[linux 网桥](https://wiki.archlinux.org/index.php/Network_bridge#Wireless_interface_on_a_bridge)（可以看成是一个虚拟网络交换机）的。因此，使用Tap使我们能够创建桥接模式的网络，让虚拟机网卡、其他虚拟机网卡以及宿主机网卡桥接在同一个网桥上面形成一个广播域，这样就能使虚拟机与虚拟机、虚拟机和宿主机以及虚拟机和外部网络之间能够相互访问了。值得注意的是，这个时候网卡是混杂模式，虚拟机网卡也会直接暴露在网络之中，会面临网络攻击的风险。另外，Tap 的网络性能也要比 user mode network 高很多。最简单的桥接配置，就是使用如下虚拟机网络的设置和删除脚本。
虚拟机启动之前，运行　`qemu-ifup`　来配置网桥和网卡. 启动虚拟机 `qemu-system-x86_64 -m 2048 -hda ubuntubase.qcow2 -smp 4 -net nic -net tap,ifname=tap0,script=no``。 `eth1` 是你的有线网卡，但是不同电脑名称不一样，我的叫 `enp3s0`, 无线网卡叫 `wlp2s0`  首先安装 `apt-get install openvpn & apt-get install bridge-utils`.
```sh
#qemu-ifup
/sbin/ifconfig eth1 down
/sbin/ifconfig eth1 0.0.0.0 promisc up
openvpn --mktun --dev tap0
ifconfig tap 0 0.0.0.0 up
brctl addbr br0
brctl addif br0 eth1
brctl addif br0 tap0
brctl stp br0 off
ifconfig br0 10.10.10.2 netmask 255.255.255.0
```
进入虚拟机设置 `sudo ifconfig eth0 10.10.10.100 netmask 255.255.255.0`. 停止虚拟机后　运行 `qemu-ifdown` 恢复网络配置。
```sh
#qemu-ifdown
ifconfig eth1 down
ifconfig eth1 -promisc
ifup eth1
ifconfig br0 down
brctl delbr br0
openvpn --rmtun --dev tap0
```


**桥接网络的设置需要注意的是，如果你是在你个人电脑上，而且还是用的是无线网络，没有使用有线网卡的话，是无效的！**
> Bridged networking works fine between a wired interface (Eg. eth0), and it is easy to setup. However if the host gets connected to the network through a wireless device, then bridging is not possible.

要想在无线网卡模式下建立桥接网络，可以给Tap设备设置静态IP,然后通过路由表来处理。可以参考[Internet sharing](https://wiki.archlinux.org/index.php/Internet_sharing)
> One way to overcome that is to setup a tap device with a static IP, making linux automatically handle the routing for it, and then forward traffic between the tap interface and the device connected to the network through iptables rules.

可以参考[Documentation/Networking/NAT](https://wiki.qemu.org/Documentation/Networking/NAT)

```sh
#!/bin/sh
#
# Copyright IBM, Corp. 2010  
#
# Authors:
#  Anthony Liguori <aliguori@us.ibm.com>
#
# This work is licensed under the terms of the GNU GPL, version 2.  See
# the COPYING file in the top-level directory.

# Set to the name of your bridge
BRIDGE=br0

# Network information
NETWORK=192.168.53.0
NETMASK=255.255.255.0
GATEWAY=192.168.53.1
DHCPRANGE=192.168.53.2,192.168.53.254

# Optionally parameters to enable PXE support
TFTPROOT=
BOOTP=

do_brctl() {
    brctl "$@"
}

do_ifconfig() {
    ifconfig "$@"
}

do_dd() {
    dd "$@"
}

do_iptables_restore() {
    iptables-restore "$@"
}

do_dnsmasq() {
    dnsmasq "$@"
}

check_bridge() {
    if do_brctl show | grep "^$1" > /dev/null 2> /dev/null; then
	return 1
    else
	return 0
    fi
}

create_bridge() {
    do_brctl addbr "$1"
    do_brctl stp "$1" off
    do_brctl setfd "$1" 0
    do_ifconfig "$1" "$GATEWAY" netmask "$NETMASK" up
}

enable_ip_forward() {
    echo 1 | do_dd of=/proc/sys/net/ipv4/ip_forward > /dev/null
}

add_filter_rules() {
do_iptables_restore <<EOF
# Generated by iptables-save v1.3.6 on Fri Aug 24 15:20:25 2007
*nat
:PREROUTING ACCEPT [61:9671]
:POSTROUTING ACCEPT [121:7499]
:OUTPUT ACCEPT [132:8691]
-A POSTROUTING -s $NETWORK/$NETMASK -j MASQUERADE 
COMMIT
# Completed on Fri Aug 24 15:20:25 2007
# Generated by iptables-save v1.3.6 on Fri Aug 24 15:20:25 2007
*filter
:INPUT ACCEPT [1453:976046]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [1605:194911]
-A INPUT -i $BRIDGE -p tcp -m tcp --dport 67 -j ACCEPT 
-A INPUT -i $BRIDGE -p udp -m udp --dport 67 -j ACCEPT 
-A INPUT -i $BRIDGE -p tcp -m tcp --dport 53 -j ACCEPT 
-A INPUT -i $BRIDGE -p udp -m udp --dport 53 -j ACCEPT 
-A FORWARD -i $1 -o $1 -j ACCEPT 
-A FORWARD -s $NETWORK/$NETMASK -i $BRIDGE -j ACCEPT 
-A FORWARD -d $NETWORK/$NETMASK -o $BRIDGE -m state --state RELATED,ESTABLISHED -j ACCEPT 
-A FORWARD -o $BRIDGE -j REJECT --reject-with icmp-port-unreachable 
-A FORWARD -i $BRIDGE -j REJECT --reject-with icmp-port-unreachable 
COMMIT
# Completed on Fri Aug 24 15:20:25 2007
EOF
}

start_dnsmasq() {
    do_dnsmasq \
	--strict-order \
	--except-interface=lo \
	--interface=$BRIDGE \
	--listen-address=$GATEWAY \
	--bind-interfaces \
	--dhcp-range=$DHCPRANGE \
	--conf-file="" \
	--pid-file=/var/run/qemu-dnsmasq-$BRIDGE.pid \
	--dhcp-leasefile=/var/run/qemu-dnsmasq-$BRIDGE.leases \
	--dhcp-no-override \
	${TFTPROOT:+"--enable-tftp"} \
	${TFTPROOT:+"--tftp-root=$TFTPROOT"} \
	${BOOTP:+"--dhcp-boot=$BOOTP"}
}

setup_bridge_nat() {
    if check_bridge "$1" ; then
	create_bridge "$1"
	enable_ip_forward
	add_filter_rules "$1"
	start_dnsmasq "$1"
    fi
}

setup_bridge_vlan() {
    if check_bridge "$1" ; then
	create_bridge "$1"
	start_dnsmasq "$1"
    fi
}

setup_bridge_nat "$BRIDGE"

if test "$1" ; then
    do_ifconfig "$1" 0.0.0.0 up
    do_brctl addif "$BRIDGE" "$1"
fi
```

### Host-Only
在Host-Only模式下，虚拟网络是一个全封闭的网络，它唯一能够访问的就是主机。其实Host-Only网络和NAT网络很相似，不同的地方就是Host-Only网络没有NAT服务，所以虚拟网络不能连接到Internet。主机和虚拟机之间的通信是通过VMware Network Adepter VMnet1虚拟网卡来实现的。
Host-Only的宗旨就是建立一个与外界隔绝的内部网络，来提高内网的安全性。这个功能或许对普通用户来说没有多大意义，但大型服务商会常常利用这个功能。如果你想为VMnet1网段提供路由功能，那就需要使用RRAS，而不能使用XP或2000的ICS，因为ICS会把内网的IP地址改为192.168.0.1，但虚拟机是不会给VMnet1虚拟网卡分配这个地址的，那么主机和虚拟机之间就不能通信了。

## VDE networking

## Socket redirection

## 参考
 - [Documentation/Networking](https://wiki.qemu.org/Documentation/Networking#Network_Basics)
 - [Configuring Guest Networking](https://www.linux-kvm.org/page/Networking)
 - [Qemu KVM 虚拟机通过虚拟网桥实现桥接和NAT的实验](http://blog2.jcix.top/2016-12-30/qemu_bridge/)

# QEMU monitor 协议(HMP/QMP)
[Qemu monitor](https://wiki.archlinux.org/index.php/QEMU#QEMU_Monitor) 就是提供了一个控制台，可以在Qemu 虚拟机运行的时候，和虚拟机交互，获取虚拟机当前信息，给虚拟机发送命令执行等。实际上 qemu monitor 设计了一种协议，可以该协议发送消息给Qemu进程，从而完成对Qemu进程的某些控制。标准控制台stdio只是其中一种实现该协议的交互方式，我们还可以让Qemu监听某个TCP端口，然后我们通过程序控制发送命令给Qemu,只要遵守QMP协议即可。
## stdio方式
参数使用 `-monitor stdio`即可。
```
sudo qemu-system-x86_64 -m 2048 -hda ubuntubase.qcow2 -smp 4 -netdev user,id=network-ubuntubase -device e1000,netdev=network-ubuntubase,mac=52:54:e4:99:78:31 -monitor stdio
```
## telnet方式
参数使用 `-monitor telnet:127.0.0.1:port,server,nowait`. 然后就可以通过 `telnet 127.0.0.1 port` 连接了。
```
sudo qemu-system-x86_64 -m 2048 -hda ubuntubase.qcow2 -smp 4 -netdev user,id=network-ubuntubase -device e1000,netdev=network-ubuntubase,mac=52:54:e4:99:78:31 -monitor telnet:127.0.0.1:9999,server,nowait
```
## unix ｓocket方式
参数使用 `-monitor unix:/tmp/monitor.sock,server,nowait`. 然后用　`socat - UNIX-CONNECT:/tmp/monitor.sock` 或者 `nc -U /tmp/monitor.sock`　或者直接写一个程序采用连接该套接字。

## tcp方式
参数使用 `-monitor tcp:127.0.0.1:port,server,nowait`, 然后使用 `nc 127.0.0.1 port`　就可以交互了，或者直接写一个采用TCP连接到该端口。


# qemu 镜像和快照管理
虚拟机也有磁盘，虚拟机的磁盘在宿主机上就是一个文件，我们一般称之为磁盘镜像，里面有虚拟机的操作系统和驱动等重要文件（对于虚拟机来说）.你可以自己制作镜像，也可以直接使用已经制作好的带操作系统的镜像。qemu 有下面的命令可以做镜像相关操作。
|command|function|
|---:|:---|
|info |查看镜像的信息|
|create|创建镜像|
|check|检查镜像|
|convert|转化镜像的格式，(raw，qcow ……)|
|snapshot|管理镜像的快照|
|rebase|在已有的镜像的基础上创建新的镜像|
|resize|增加或减小镜像大小|

## 外置快照
### 利用BlockCommit 进行快照回滚
```
.------------.  .------------.  .------------.  .------------.  .------------.
|            |  |            |  |            |  |            |  |            |
| RootBase   <---  Snap-1    <---  Snap-2    <---  Snap-3    <---  Snap-4    |
|            |  |            |  |            |  |            |  | (Active)   |
'------------'  '------------'  '------------'  '------------'  '------------'
                                 /                  |
                                /                   |
                               /  commit data       |
                              /                     |
                             /                      |
                            /                       |
                           v           commit data  |
.------------.  .------------. <--------------------'           .------------.
|            |  |            |                                  |            |
| RootBase   <---  Snap-1    |<---------------------------------|  Snap-4    |
|            |  |            |       Backing File               | (Active)   |
'------------'  '------------'                                  '------------'

```

### 利用在线快照制作在线的镜像模板(镜像模板保存了磁盘当前所有状态数据，可以用来恢复或者创建新的虚拟机使用)

下面这个原理图，可以用来制作磁盘模板. 就是通过产生一个快照，然后将RootBase合并到当前激活快照中，形成一个新镜像模板。
```
.------------.  .------------.  .------------.  .------------.  .------------.
|            |  |            |  |            |  |            |  |            |
| RootBase   <---  Snap-1    <---  Snap-2    <---  Snap-3    <---  Snap-4    |
|            |  |            |  |            |  |            |  | (Active)   |
'------------'  '------------'  '------------'  '------------'  '------------'
      |                  |              |                  \
      |                  |              |                   \
      |                  |              |                    \  stream data
      |                  |              | stream data         \
      |                  |              |                      \
      |                  | stream data  |                       \
      |  stream data     |              '------------------>     v
      |                  |                                    .--------------.
      |                  '--------------------------------->  |              |
      |                                                       |  Snap-4      |
      '---------------------------------------------------->  | (Active)     |
                                                              '--------------'
                                                                'Standalone'
                                                                (w/o backing
                                                                file)

```
查看当前虚拟机使用的块设备。
```sh
(qemu) info block
ide0-hd0 (#block121): ubuntu1.qcow2 (qcow2)
    Cache mode:       writeback
    Backing file:     ubuntubase.qcow2 (chain depth: 1)

ide1-cd0: [not inserted]
    Removable device: not locked, tray closed

floppy0: [not inserted]
    Removable device: not locked, tray open

sd0: [not inserted]
    Removable device: not locked, tray closed

```
在线情况下给虚拟机做一个外置快照, `snapshot_blkdev <block-device> <snapshot-name> <format>`
```sh
(qemu) snapshot_blkdev ide0-hd0 ubuntu1-snapshot.img qcow2
Formatting 'ubuntu1-snapshot.img', fmt=qcow2 size=21474836480 backing_file=ubuntu1.qcow2 backing_fmt=qcow2 encryption=off cluster_size=65536 lazy_refcounts=off refcount_bits=16
(qemu) info block
ide0-hd0 (#block1469): ubuntu1-snapshot.img (qcow2)
    Cache mode:       writeback
    Backing file:     ubuntu1.qcow2 (chain depth: 2)

ide1-cd0: [not inserted]
    Removable device: not locked, tray closed

floppy0: [not inserted]
    Removable device: not locked, tray open

sd0: [not inserted]
    Removable device: not locked, tray closed

```
将快照(镜像)的backing_file合并到当前的外置快照之中，从而得到一个模板。 `block_stream <block-device>`
```sh
(qemu) block_stream ide0-hd0 
(qemu) info block
ide0-hd0 (#block1469): ubuntu1-snapshot.img (qcow2)
    Cache mode:       writeback
    Backing file:     ubuntu1.qcow2 (chain depth: 2)

ide1-cd0: [not inserted]
    Removable device: not locked, tray closed

floppy0: [not inserted]
    Removable device: not locked, tray open

sd0: [not inserted]
    Removable device: not locked, tray closed
(qemu) info block-jobs
No active jobs
``` 

### 快照删除
当一个或多个快照不再需要时，您可以删除它们。删除快照后，您将无法把虚拟机恢复到这些快照所包括的时间点上。删除快照并不一定会获得更多的可用存储空间，而这些快照的数据也不一定会被实际删除。例如，您的虚拟机有 5 个快照，如果您删除了第 3 个快照，第 3 个快照中的数据可能仍然会存在在系统中，因为第 4 和第 5 个快照可能会需要这些数据。一般情况下，删除快照通常可以提高虚拟机的性能。
当选择了一个要被删除的快照时，QEMU 会创建一个和要被删除的快照大小相同的新逻辑卷。被删除的快照会和它随后的那个快照进行合并，而新建逻辑卷的大小会被扩充来保存快照合并的结果。如果被删除的快照和它随后的那个快照没有重叠的内容，这个新建逻辑卷的大小将和被合并的两个快照的大小总和相同。当合并完成后，随后的那个快照会被改名，并被标记为已被删除，而用来保存合并结果的逻辑卷会使用它的名字来替代这个快照。最终删除的结果是选择被删除的快照和它随后的那个快照都会标记为已被删除，而这两个快照的合并结果会作为一个快照来取代它们的位置。
例如，快照 Delete_snapshot 的大小是 200GB，它随后的那个快照（Next_snapshot）的大小是 100GB。Delete_snapshot 被删除，一个新的逻辑卷（临时命名为 Snapshot_merge）会被创建，它的初始大小是 200GB。然后，Snapshot_merge 的大小会被扩展到 300GB 来保存 Delete_snapshot 和 Next_snapshot 的合并结果。随后，Next_snapshot 会被改名为 Delete_me_too_snapshot，而 Snapshot_merge 被重新命名为 Next_snapshot。最后，Delete_snapshot 和 Delete_me_too_snapshot 都被删除。

<img src="/static/images/Virtualization/qemu-kvm/qemu-snapshot-delete.png" style="width:500px;height:300px;">
<caption><center><u> <font color="purple"> **snapshot-delete** </u></font> </center></caption>

### 参考
 - [qemu-snapshot](http://m.udpwork.com/item/12674.html)
 - [QCOW2 backing files & overlays](https://kashyapc.fedorapeople.org/virt/lc-2012/snapshots-handout.html)
 - [Features/Snapshots](https://wiki.qemu.org/Features/Snapshots)


## 保存镜像（都是关机状态，直接对磁盘进行操作）


## 更改　qemu 镜像密码
比如我制作了一个镜像ubuntubase.qcow2.　但是这个镜像的密码忘记了，启动虚拟机的时候无法登录，你可以通过以下两种方法改变密码。实际生产环境中，也是通过该方式重置用户自己设置的账号和密码，挂载之后其实可以直接更改 `/etc/shadow` 内容，从而重置密码。或者是部署ssh keypair
### 1. 使用内核模块 qemu-nbd 挂载修改
检查 nbd 模块是否加载，`lsmod | grep nbd` 查看有nbd，则说明已经加载了nbd 内核模块。若没有，则使用 `sudo modprobe nbd` 进行加载。操作在root权限下进行

1. 通过 qemu-nbd 连接镜像到块设备 `qemu-nbd -c /dev/nbd0 ubuntubase.qcow2`
2. 挂载设备到文件系统 `mount /dev/nbd0p1 /mnt`
3. 采用 [chroot](http://man.linuxde.net/chroot) 更换根目录，这样就可以更改镜像系统的root 密码了，因为整个系统会认为是在以 /mnt　为根目录的新系统里　`chroot /mnt`
4. passwd 修改密码,这个时候修改的文件就是以 /mnt 为根目录的某些密码文件了. 更改用户tq的密码
```sh
root@kinny-Lenovo-XiaoXin:/# passwd tq
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
```
5. 卸载设备 `umount /mnt`
6. 断开块设备连接 `qemu-nbd -d /dev/nbd0p1`

### 2. 使用 [libguestfs](http://libguestfs.org/)挂载修改
安装
```sh
sudo yum install libguestfs-tools      # Fedora/RHEL/CentOS
sudo apt-get install libguestfs-tools  # Debian/Ubuntu
guestfish --ro -i -a disk.img
```
然后通过直接修改密码文件的方式修改密码. 先使用 openssl 产生加密后的密码串。
```
openssl passwd -1 new_password
```
然后将密码串替换原来的root密码
```
guestfish -a ubuntu-server.qcow2
><fs> run
><fs> list-filesystems
><fs> mount /dev/sda1 /
><fs> vi /etc/shadow
><fs> quit
```
### 参考
 - [How to reset forgotten root password for Linux KVM qcow2 image/vm](https://www.cyberciti.biz/faq/how-to-reset-forgotten-root-password-for-linux-kvm-qcow2-image-vm/)

