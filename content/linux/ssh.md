---
title: "ssh"
date: 2017-03-31 15:14
---
# 端口转发映射
第二种：使用SSH协议转发

ssh的三个强大的端口转发命令：

1. 转发到远端：`ssh -C -f -N -g -L 本地端口:目标IP:目标端口 用户名@目标IP`
2. 转发到本地：`ssh -C -f -N -g –R 本地端口:目标IP:目标端口 用户名@目标IP`
3. `ssh -C -f -N -g -D listen_port user@Tunnel_Host`
ssh -g -L 44392:localhost:22 tianqi05@10.4.225.98
端口转发 ssh -p 44392 root@10.4.225.98

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