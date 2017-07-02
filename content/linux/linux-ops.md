---
title: "linux-ops"
date: 2017-07-02 22:39
---

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