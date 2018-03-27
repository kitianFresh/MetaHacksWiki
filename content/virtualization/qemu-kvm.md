---
title: "qemu-kvm"
date: 2018-03-27 17:10
---


# 虚拟机磁盘镜像
虚拟机也有磁盘，虚拟机的磁盘在宿主机上就是一个文件，我们一般称之为磁盘镜像，里面有虚拟机的操作系统和驱动等重要文件（对于虚拟机来说）.你可以自己制作镜像，也可以直接使用已经制作好的带操作系统的镜像。

# 更改镜像密码
比如我下载了一个镜像centosbase.qcow2.　但是这个镜像的密码忘记了，启动虚拟机的时候无法登录，你可以通过以下两种方法改变密码。
## 挂载镜像到宿主机通过chroot 修改
检查 nbd 模块是否加载，`lsmod | grep nbd` 查看有nbd，则说明已经加载了nbd 内核模块。若没有，则使用 `sudo modprobe nbd` 进行加载。操作在root权限下进行

1. 通过 qemu-nbd 连接镜像到块设备 `qemu-nbd -c /dev/nbd0 centosbase.qcow2`
2. 挂载设备到文件系统 `mount /dev/nbd0p1 /mnt`
3. 采用 [chroot](http://man.linuxde.net/chroot) 更换根目录，这样就可以更改镜像系统的root 密码了，因为整个系统会认为是在以 /mnt　为根目录的新系统里　`chroot /mnt`
4. passwd 修改密码,这个时候修改的文件就是以 /mnt 为根目录的某些密码文件了
```shmbash-4.2# 
bash-4.2# passwd
Changing password for user root.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: all authentication tokens updated successfully.
bash-4.2# 
bash-4.2# exit
```
5. 卸载设备 `umount /mnt`
6. 断开块设备连接 `qemu-nbd -d /dev/nbd0p1`

## 使用 [libguestfs](http://libguestfs.org/)
```sh
sudo yum install libguestfs-tools      # Fedora/RHEL/CentOS
sudo apt-get install libguestfs-tools  # Debian/Ubuntu
guestfish --ro -i -a disk.img
```

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

### 制作虚拟机镜像
qemu-img create -f raw /images/vm1.raw 8G

压缩办法：
qemu-img convert -c -O qcow2 /dev/shm/win.qcow2 /home/soft/kvm/ocr.qcow2
其中ocr.qcow2是你的目标镜像

转换办法(转换即压缩)：
qemu-img convert -f raw centos.img -O qcow2 centos.qcow2
参数说明：convert 将磁盘文件转换为指定格式的文件-f 指定需要转换文件的文件格式-O 指定要转换的目标格式转换完成后，将新生产一个目标映像文件，原文件仍保存。

### 保存镜像


# qemu 快照原理


# qemu monitor 协议(HMP/QMP)