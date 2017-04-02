---
title: "ssh"
date: 2017-03-31 15:14
---

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