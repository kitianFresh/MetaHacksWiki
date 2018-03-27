---
title: "qemu-kvm"
date: 2018-03-27 17:10
---


# 虚拟机磁盘镜像
虚拟机也有磁盘，虚拟机的磁盘在宿主机上就是一个文件，我们一般称之为磁盘镜像，里面有虚拟机的操作系统和驱动等重要文件（对于虚拟机来说）.

## qemu-img 镜像管理
- info
查看镜像的信息
- create
创建镜像
- check
检查镜像
- convert
转化镜像的格式，（raw，qcow ……）
- snapshot
管理镜像的快照
- rebase
在已有的镜像的基础上创建新的镜像
- resize
增加或减小镜像大小

### 生成虚拟机镜像
qemu-img create -f raw /images/vm1.raw 8G

压缩办法：
qemu-img convert -c -O qcow2 /dev/shm/win.qcow2 /home/soft/kvm/ocr.qcow2
其中ocr.qcow2是你的目标镜像

转换办法(转换即压缩)：
qemu-img convert -f raw centos.img -O qcow2 centos.qcow2
参数说明：convert 将磁盘文件转换为指定格式的文件-f 指定需要转换文件的文件格式-O 指定要转换的目标格式转换完成后，将新生产一个目标映像文件，原文件仍保存。

# qemu 快照原理


# qemu monitor 协议(HMP/QMP)