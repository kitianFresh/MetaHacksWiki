---
title: "ssh"
date: 2017-03-31 15:14
---
[TOC]

# ssh 设置 linux 主机互信，不输入密码
## 手动方式
将机器A 的公钥 `id_rsa.pub` 加入到 机器B的 `.ssh` 目录下的 `authorized_keys` 文件中即可； 但是一般使用 ssh-keygen 命令使用默认设置产生的密钥对`.ssh`目录里面没有 `authorized_keys`, 此时需要手动新建，这个时候很容易出错，新建之后，你会发现每次使用ssh登陆还是需要密码，重启 sshd 服务也不管用。

`ssh -v user@host` 去查看登陆过程的异常；
这是因为 sshd 这个进程对 `authorized_keys` 有权限限制，如果`authorized_keys`文件、`$HOME/.ssh`目录 或 `$HOME`目录让本用户之外的用户有写权限，系统用户的ssh 在 `/etc/ssh/ssh_known_hosts` 目录下，那么sshd都会拒绝使用 `~/.ssh/authorized_keys` 文件中的key来进行认证的。因此你需要使用 `chmod 644 authorized_keys` 来修改文件读写权限，只有当前用户有写权限，其他只有读权限；使用 `man sshd` 查看相关手册；
```sh
~/.ssh/authorized_keys
             Lists the public keys (RSA/ECDSA/DSA) that can be used for logging in as this user.  The format
             of this file is described above.  The content of the file is not highly sensitive, but the recom-
             mended permissions are read/write for the user, and not accessible by others.

             If this file, the ~/.ssh directory, or the user’s home directory are writable by other users,
             then the file could be modified or replaced by unauthorized users.  In this case, sshd will not
             allow it to be used unless the StrictModes option has been set to “no”.
```
## 自动方式
`ssh-copy-id user@host` 是一个自动化以上过程的公钥拷贝+修改`authorized_keys` 命令，直接就解决了；查看手册：
```sh
ssh-copy-id  is a script that uses ssh to log into a remote machine (presumably using a login password,
so password authentication should be enabled, unless you’ve done some clever use  of  multiple  identi-
ties)  It also changes the permissions of the remote user’s home, ~/.ssh, and ~/.ssh/authorized_keys to
remove group writability (which would otherwise prevent you from logging in, if  the  remote  sshd  has
StrictModes  set  in its configuration).  If the -i option is given then the identity file (defaults to
~/.ssh/id_rsa.pub) is used, regardless of whether there are any keys in your ssh-agent.  Otherwise,  if
this:       ssh-add -L provides any output, it uses that in preference to the identity file.  If the -i
option is used, or the ssh-add produced no output, then it uses the  contents  of  the  identity  file.
Once  it  has  one or more fingerprints (by whatever means) it uses ssh to append them to ~/.ssh/autho-
rized_keys on the remote machine (creating the file, and directory, if necessary)
```

# 端口转发映射
第二种：使用SSH协议转发

ssh的三个强大的端口转发命令：

1. 转发到远端：`ssh -C -f -N -g -L 本地端口:目标IP:目标端口 用户名@目标IP`
2. 转发到本地：`ssh -C -f -N -g –R 本地端口:目标IP:目标端口 用户名@目标IP`
3. `ssh -C -f -N -g -D listen_port user@Tunnel_Host`
ssh -g -L 44392:localhost:22 tianqi05@10.4.225.98
端口转发 ssh -p 44392 root@10.4.225.98
让本地机器从浏览器访问远程的 jupyter server。`ssh -L 8000:localhost:8888 ubuntu@hacksmeta.com` 这样就可以从本地浏览器的8000端口，访问服务器hacksmeta.com上8888端口的jupyter了。

# ssh
```
ssh -p user@host
ssh -p 29911 root@hillsun.hacksmeta.com
```
密钥分发
```
cat id_rsa.pub.bak | ssh kinny@ai.mc2lab.com "cat >>  ~/.ssh/authorized_keys"

scp id_rsa.pub.bak kinny@hacksmeta.com:~/.ssh
```

# scp(Security Copying)

1. 文件从本机到服务器
`scp user_item_analysis.py ustc@192.168.231.97:~/kinny/`

2. 文件从服务器下载到本地
`scp ustc@192.168.231.97:~/kinny/data/user_item_label_table.csv ./Study/competition/jd/JData/data`

3. 目录
加个 `-r` 参数 `scp -r` 

## SCP 原理其实是基于SSH的
 - [SCP原理](https://blog.xiaket.org/2010/how-scp-protocol-works.html)


# ssh 远程执行命令和脚本
### 在远程主机上执行命令
```sh
ssh user@remoteNode "cd /home ; ls"
ssh user@remoteHost "cat /proc/cpuinfo"
```

### 直接执行远程主机上的脚本
ssh 调用远程命令后不能自动退出解决方法
可以将标准输出与标准错误输出重定向到/dev/null，这样就不会一直处于等待状态。
ssh -l www-online 192.168.110.34 “/home/www-online/uptimelog.sh > /dev/null 2>&1 &”

### 让一个本地脚本在远程主机执行
以下这个脚本test.sh可以在远程主机执行的方式
```sh
ls
pwd
echo $0
echo $1
echo $2
```
`ssh user@remoteHost "bash -s" < test.sh helloword kiki`; 执行结果如下，helloword kiki 是脚本的 $1 和 $2 参数。
```sh
dockerapp
py2virtualenv
qemu-2.11.0-rc0.tar.xz
test_encoding.py
uts.c
work
/home/ubuntu
bash
helloworld
kiki
```