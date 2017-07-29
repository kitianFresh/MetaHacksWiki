---
title: "CentosTips"
date: 2017-07-29 18:36
---
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

# rpm删除安装
```
rpm -qal | grep 
sudo rpm -e xx.rpm
sudo rpm -ivh XX.rpm
```

系统中的awk命令到底是执行哪个可以执行文件呢？
```
$ readlink /usr/bin/awk  
/etc/alternatives/awk  ----> 其实这个还是一个符号连接  
$ readlink /etc/alternatives/awk  
/usr/bin/gawk  ----> 这个才是真正的可执行文件  
-f 选项：
-f 选项可以递归跟随给出文件名的所有符号链接以标准化，除最后一个外所有组件必须存在。
简单地说，就是一直跟随符号链接，直到直到非符号链接的文件位置，限制是最后必须存在一个非符号链接的文件。
$ readlink -f /usr/bin/awk  
/usr/bin/gawk  
```
