---
title: "virtualbox"
date: 2018-01-04 20:55
---

# 后台启动VM
VBoxManage startvm <uuid>|<name> [--type gui|sdl|headless] 使用headless 选项，无gui的服务器版本

# 查看VM
vboxmanage list vms
vboxmanage list running vms

# 控制VM
vboxmanage controlvm <uuid>|<vname> pause|resume|reset|poweroff|savestate

# 设置共享目录

如果想使用共享目录则：
vboxmanage sharedfolder add centos --name *share* --hostpath *~/share*

登录成功后挂载共享目录:
mount -t vboxsf share /mnt/