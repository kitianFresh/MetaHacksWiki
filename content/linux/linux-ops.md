---
title: "linux-ops"
date: 2017-07-02 22:39
---
[TOC]

# Linux 软件源更新
更改之前注意备份: `sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup`

```
deb http://cn.archive.ubuntu.com/ubuntu/ willy main restricted universe multiverse
deb http://cn.archive.ubuntu.com/ubuntu/ willy-security main restricted universe multiverse
deb http://cn.archive.ubuntu.com/ubuntu/ willy-updates main restricted universe multiverse
deb http://cn.archive.ubuntu.com/ubuntu/ willy-backports main restricted universe multiverse
##测试版源
deb http://cn.archive.ubuntu.com/ubuntu/ willy-proposed main restricted universe multiverse
# 源码
deb-src http://cn.archive.ubuntu.com/ubuntu/ willy main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ willy-security main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ willy-updates main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ willy-backports main restricted universe multiverse
##测试版源
deb-src http://cn.archive.ubuntu.com/ubuntu/ willy-proposed main restricted universe multiverse
```
更改的话, 第一个是更改地址, 如阿里云的,  第二个是更改代号即版本, 如 willy(15.10) 修改成 xenial(16.04); 

[参考](http://wiki.ubuntu.org.cn/%E6%BA%90%E5%88%97%E8%A1%A8)

# OpenStack 安装问题

1. `generate-subunit: command not found`
```
sudo pip install -U os-testr
```

2. `/bin/sh: 1: brctl: not found`
```
sudo apt-get install bridge-utils
```

3. 
```
git config --global url."https://".insteadOf git://
```

- [The Hitchhiker’s Guide to Python!](http://docs.python-guide.org/en/latest/)
- [OpenStack Hacker养成指南](https://www.ustack.com/blog/openstack_hacker/#i)
- [用DevStack安装OpenStack(单机)](http://www.ganecheng.tech/blog/53538203.html)
- [devstack 安装教程](http://chbrian.github.io/cloud/2014/02/22/devstack-/)
- [openstack](https://docs.openstack.org/mitaka/zh_CN/install-guide-ubuntu/environment.html)


# 配置 sudo 用户.
首先使用 `useradd username` 添加新用户，然后设置用户密码 `passwd usernmae`.
然后 使用 `visudo` 自动打开 `/etc/sudoers` 配置文件，在文件加入sudo权限，配置如下：
```
tianqi05    ALL=(ALL)    ALL
‘不要密码
tianqi05    ALL=(ALL)    NOPASSWD:ALL
```
最后切换用户
sudo -iu tianqi05


# screen 使用简介
screen 和 tmux有类似的服务。只是screen开启会话后不会新建窗口，还是在当前窗口。

1. screen 创建一个session
```
screen -S myNewSession
或者
screen 创建一个名字默认是ttys[ID] 的session.
```
2. 从当前 session 暂时断开退出。 `ctrl+a+d`

3. 列出当前所有的session. `screen -ls` 
```
There are screens on:
	28007.ttys005.tianqideMacBook-Pro	(Detached)
	28091.myNewSession	(Detached)
2 Sockets in /var/folders/bb/ywfc1vbs5lx_blqpl5ssrvv40000gp/T/.screen.
```
4. 恢复某一个session。`screen -r myNewSession`

5. 强制杀死一个 session. `kill -9 28091`
```
There are screens on:
	28007.ttys005.tianqideMacBook-Pro	(Detached)
	28091.myNewSession	(Dead ???)
Remove dead screens with 'screen -wipe'.
2 Sockets in /var/folders/bb/ywfc1vbs5lx_blqpl5ssrvv40000gp/T/.screen.
```
6. 按照提示清理。 `screen -wipe`

7. `Cannot open your terminal '/dev/pts/0' - please check.`
1. Sign out and properly connect / sign in as the user you wish to use.
2. Run `script /dev/null` to own the shell (more info over at Server Fault); then try screen again.

# 配置 iTerm2 和 tmux

Tmux 是 Terminal multiplexer 的缩写， 其实就是 终端可以复用的意思。 

场景就是，第一种是在本地电脑，你打开了好几个terminal， 每一个terminal由当前的工作环境， 然后这个时候没电了或者你电脑关机了，那么这些terminal的工作状态就没有了。如果使用tmux记录下来，那么这些terminal下次还可以继续使用，并且恢复到关闭时的状态。

第二种就是 ssh 登陆到远程主机， 然后ssh 基本上很容易断开，一般使用 nohup & 在后台运行命令， 但是如果使用 tmux， 就可以记录下来，关闭了也没关系，还可以再次恢复。

执行 tmux -CC 之后会开启一个 session， tmux -CC 默认会以数字编号命名一个回话session， 并开启一个新的iterm窗口，在这个窗口即回话下面， 使用 command+T 可以创建多个窗口（注意不是command + D, 也不是 command + N）， 原来 tmux -CC 创建的窗口会阻塞，可以通过 esc 退出， 然后再通过 tmux -CC attach 恢复， 但是 tmux attach 恢复出来是窗口叠加在一起的， 加 CC 参才会原样恢复。

另外， tmux new -s session\_name, 可以给一个session起名字， 以后可以使用 tmux attach -t session\_name 恢复某个会话。

## 局域网下互传大文件

 解决方案很多，第一种不通过中间服务器， 直接两台PC之间互传，如果在同一个局域网之下， 确保可以相互ping通或者Telnet。一台机器充当服务器，另一台客户机， 采用 Python 自带的SimpleHTTPServer即可。客户端 `wget http://[IP]:[port]/[path\_to\_file]`; 另外，下载某个目录下的所有文件，`wget -r -np -nH -R index.html http://url/including/files/you/want/to/download/`. `-r` 递归， `-R index.html` 不带 `index.html` 文件。



## ifconfig VS ip

 - [ifconfig vs ip: What’s Difference and Comparing Network Configuration](https://www.tecmint.com/ifconfig-vs-ip-command-comparing-network-configuration/)



 ## 查看进程所暂用的资源
 首先找到进程 `ps aux|grep XXX`, 然后使用 `sudo lsof -p [pid]` 查看该进程占用的所有资源；也可以查看某个资源被哪些进程占用， `lsof [filename]` 可以看到进程pid， 拿到pid 之后可以使用`ps aux|grep [pid]` 查看是什么进程；




## 删除某个目录下的大量的文件
```sh
find /tmp -name core -type f -print0 | xargs -0 /bin/rm -f
```
Find files named core in or below the directory /tmp and delete them, processing filenames in such a  way
that file or directory names containing spaces or newlines are correctly handled.
```
find /tmp -depth -name core -type f -delete
```
Find  files  named  core in or below the directory /tmp and delete them, but more efficiently than in the
previous example (because we avoid the need to use fork(2) and exec(2) to launch rm and we don't need the
extra xargs process).
```
cut -d: -f1 < /etc/passwd | sort | xargs echo
```
/bin/rm: Argument list too long.
The problem is that when you type something like “rm -rf *”, the “*” is replaced with a list of every matching file, like “rm -rf file1 file2 file3 file4” and so on. There is a reletively small buffer of memory allocated to storing this list of arguments and if it is filled up, the shell will not execute the program.
To get around this problem, a lot of people will use the find command to find every file and pass them one-by-one to the “rm” command like this:
find . -type f -exec rm -v {} \;
My problem is that I needed to delete 500,000 files and it was taking way too long.
I stumbled upon a much faster way of deleting files – the “find” command has a “-delete” flag built right in! Here’s what I ended up using:
find . -type f -delete
Using this method, I was deleting files at a rate of about 2000 files/second – much faster!
You can also show the filenames as you’re deleting them:
find . -type f -print -delete
…or even show how many files will be deleted, then time how long it takes to delete them:

ls -1 汇报内存不足如果目录下有上百万个小文件
ls -1 | wc -l && time find . -type f -delete

 - [目录中文件过多导致ls命令卡住-利用空设备重定向和strace监控ls](https://www.jianshu.com/p/353a5dbcd423)

# check dev file system
1. `lsblk -f` 可以看到所有附着设备的文件系统类型，如果有的话；无论是否mount
2. `df -Th` 可以看到所有已经mount的设备的文件系统类型
3. `fsck -N /dev/vdc1` 设备修复工具，可以对某个分区进行修复，并可以查看到该分区的文件系统
4. `mount | grep "^/dev"`
5. `blkid /dev/sda1`
6. `file -sL /dev/sda1`
7. `cat /etc/fstab` 在主机启动的时候回挂载改文件内容中的设备，里面描述了该挂载的设备的id，tag，label，mount point， fs等，但是这个是手动设置的

# 检查占用空间大的文件
```
sudo find . -type f -size +1000M  -print0 | xargs -0 du -h | sort -nr
```


# 大文件分块计算md5 的命令
例如 5M 一块，可以设置块大小1M, 一次读入5个就是可以当成读一大块5M了，然后skip可以设置偏移量。比如 14M 可以如下操作，分三块读。
```python
dd bs=1m count=5 skip=0 if=someFile | md5 >>checksums.txt

dd bs=1m count=5 skip=5 if=someFile | md5 >>checksums.txt

dd bs=1m count=5 skip=10 if=someFile | md5 >>checksums.txt
# 最后计算连接的md5
cat checksums.txt | md5
```

# Linux分区和增大交换区
 - [How to increase swap space?](https://askubuntu.com/questions/178712/how-to-increase-swap-space/389067#389067)

 - [ How To Add Swap Space on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-16-04)

# Linux 磁盘性能测试benchmark
## dd

1. write test; `sync; dd if=/dev/zero of=tempfile bs=1M count=1024; sync`

2. clear memory cache for tempfile; `sudo /sbin/sysctl -w vm.drop_caches=3`

3. read test; `dd if=tempfile of=/dev/null bs=1M count=1024`

## hdparm
sudo hdparm -Tt /dev/sdb1

## iotop


 - [Disk Speed Test (Read/Write): HDD, SSD Performance in Linux](https://www.shellhacks.com/disk-speed-test-read-write-hdd-ssd-perfomance-linux/)