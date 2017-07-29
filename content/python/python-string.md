---
title: "python-string"
date: 2017-04-30 11:22
---

## 格式字符串手册
数字格式化

下面的表格展示了使用Python的后起新秀str.format()格式化数字的多种方法，包含浮点数格式化与整数格式化示例。可使用 print("FORMAT".format(NUMBER)); 来运行示例，因此你可以运行： print("{:.2f}".format(3.1415926)); 来得到第一个示例的输出。

        数字	格式	输出	描述
        3.1415926	{:.2f}	3.14	保留小数点后两位
        3.1415926	{:+.2f}	+3.14	带符号保留小数点后两位
        -1	{:+.2f}	-1.00	带符号保留小数点后两位
        2.71828	{:.0f}	3	不带小数
        5	{:0>2d}	05	数字补零 (填充左边, 宽度为2)
        5	{:x<4d}	5xxx	数字补x (填充右边, 宽度为4)
        10	{:x<4d}	10xx	数字补x (填充右边, 宽度为4)
        1000000	{:,}	1,000,000	以逗号分隔的数字格式
        0.25	{:.2%}	25.00%	百分比格式
        1000000000	{:.2e}	1.00e+09	指数记法
        13	{:10d}	        13	右对齐 (默认, 宽度为10)
        13	{:<10d}	13	左对齐 (宽度为10)
        13	{:^10d}	    13	中间对齐 (宽度为10)



## Python获取application运行的路径
```python
import sys

def is_packaged():
    if hasattr(sys, 'frozen'):
        return sys.frozen

    return False


def get_root():
    return sys._MEIPASS

```
```python

from . import pyinstaller


def get_mclouds_root():
    if pyinstaller.is_packaged():
        return pyinstaller.get_root()

    cur_path = os.path.dirname(__file__)
    cur_path = os.path.abspath(cur_path)
    assert(cur_path.endswith('clouds/common'))
    pos = cur_path.rfind('meituan/server')
    assert(pos > 0)

    return cur_path[:pos]


def get_mclouds_scripts_path():
    return os.path.join(get_mclouds_root(), 'meituan/scripts')


def get_mclouds_workspace_path():
    root_path = get_mclouds_root()
    if pyinstaller.is_packaged():
        root_path = '/opt/cloud/'

    return os.path.join(root_path, 'workspace')
```
## chroot机制可以更改当前程序运行的参考根目录
chroot 可以实现安全隔离，从新制造一个新的根目录，并且程序只对该目新的根目录具有权限，其他目录对该进程就不可见了。从而实现安全隔离，但是如果程序还想要访问其他的库，命令，需要组织根目录，把相应的库文件放到相应的目录下面，可以用 busybox实现这个功能。就是有制造出一个linux系统资源目录出来给某个进程使用，类似容器隔离技术。
```python
import os, sys
newroot = '/Users/kiki/tmp'
if not os.path.exists(newroot):
    os.makedirs(newroot)
os.chroot(newroot)
print __file__
print os.getcwd()
print os.path.abspath(__file__)
```
## argparse 中 `metavar` 不能和`action=store_true` 同时用。
