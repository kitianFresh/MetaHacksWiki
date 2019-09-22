---
title: "cgroup"
date: 2019-08-22 17:47
---

- [brendangregg/dockerpsns.sh](https://gist.github.com/brendangregg/1abcfeef9155ac526197f6f0abdd86bf)

```bash
#!/bin/bash
#
# dockerpsns - proof of concept for a "docker ps --namespaces".
#
# USAGE: ./dockerpsns.sh
#
# This lists containers, their init PIDs, and namespace IDs. If container
# namespaces equal the host namespace, they are colored red (this can be
# disabled by setting color=0 below).
#
# Copyright 2017 Netflix, Inc.
# Licensed under the Apache License, Version 2.0 (the "License")
#
# 10-Apr-2017   Brendan Gregg   Created this.

namespaces="cgroup ipc mnt net pid user uts"
color=1
declare -A hostns

printf "%-14s %-20s %6s %-16s" "CONTAINER" "NAME" "PID" "PATH"
for n in $namespaces; do
    printf " %-10s" $(echo $n | tr a-z A-Z)
done
echo

# print host details
pid=1
read name < /proc/$pid/comm
printf "%-14s %-20.20s %6d %-16.16s" "host" $(hostname) $pid $name
for n in $namespaces; do
    id=$(stat --format="%N" /proc/$pid/ns/$n)
    id=${id#*[}
    id=${id%]*}
    hostns[$n]=$id
    printf " %-10s" "$id"
done
echo

# print containers
for UUID in $(docker ps -q); do
    # docker info:
    pid=$(docker inspect -f '{{.State.Pid}}' $UUID)
    name=$(docker inspect -f '{{.Name}}' $UUID)
    path=$(docker inspect -f '{{.Path}}' $UUID)
    name=${name#/}
    printf "%-14s %-20.20s %6d %-16.16s" $UUID $name $pid $path

    # namespace info:
    for n in $namespaces; do
        id=$(stat --format="%N" /proc/$pid/ns/$n)
        id=${id#*[}
        id=${id%]*}
        docolor=0
        if (( color )); then
            [[ "${hostns[$n]}" == "$id" ]] && docolor=1
        fi
        (( docolor )) && echo -e "\e[31;1m\c"
        printf " %-10s" "$id"
        (( docolor )) && echo -e "\e[0m\c"
    done
    echo
    
done
```

# systemd
systemd 用于并行化启动服务，来加快服务启动；linux 1 号进程经历了很多
 -[LINUX PID 1 和 SYSTEMD](https://coolshell.cn/articles/17998.html)

可以通过多种方式使用cgroups
1. 直接使用一些工具，cgcreate, cgexec, cgclassify
2. 使用 rules engine daemon，自动移动某些用户/组/命令 到cgroup(/etc/cgrules.conf和/usr/lib/systemd/system/cgconfig.service)
3. 通过某些虚拟化工具，比如 linux containers

上面这些本质都是通过cgroup提供的文件系统结构来操作cgroup

另外，systemd 本身采用了cgroups 来组织进程结构

# cgroup
 - [Cgroups 与 Systemd](https://www.cnblogs.com/sparkdev/p/9523194.html)
 - [从 init 系统说起](https://www.cnblogs.com/sparkdev/p/8448237.html)
 - [howto-use-cgroup](http://tiewei.github.io/devops/howto-use-cgroup/)
 - [Control groups, part 6: A look under the hood](https://lwn.net/Articles/606925/)




com.sankuai.bikebj.staging.kubemaser

各位RD大佬，辛苦帮忙进行二次确认，这些服务是否有状态，无状态服务后续能够进行容器化的自动迁移，如果必须是有状态服务，就可以不确认，但是有很多服务可以改造成无状态服务的，可以进行确认，但确认的时候注意标明改造完成时间，谢谢大家配合，截止日期周三
@所有人





