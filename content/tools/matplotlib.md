---
title: "matplotlib font set"
date: 2017-03-25 19:54
---

# matplotlib font set
ubuntu16.04 环境下直接使用　matplotlib 绘图输出中文可能会出现乱码的现象，需要设置相应的字体！ 如果没有字体以及没有配置　matplotlibrc 文件和　删除　fontList.cache　缓存，直接在代码中使用
`mpl.rcParams['font.sans-serif'] = 'simhei'` 是无效的，需要同时满足三个条件.

如果你在其他工具和包中出现需要中文字体的问题，这个思路应该也是行的通的．

## 下载中文字体
如果你要显示中文，当然中文的字体必不可少，字体在 Ubuntu 和　Windows 平台下应该是通用的，一般都使用 .ttf 后缀的文件格式.首先到[myfontfree](http://www.myfontfree.com/microsoft-yahei-myfontfreecom66f594.htm)下载chinese.msyh.ttf字体，将其拷贝到/usr/share/fonts 目录下面，但一般还不够，你需要再把字体拷贝到工具安装包相应的字体目录下面，matplotlib 具体安装目录是
```
python2.7 和　python3.5类似，就是版本变了

/usr/local/lib/python2.7/dist-packages/matplotlib/mpl-data/fonts/ttf/

/usr/local/lib/python3.5/dist-packages/matplotlib/mpl-data/fonts/ttf/
```
当然也可以使用软链接的形式,在安装包字体目录下面，`sudo ln -s /usr/share/font/simhei.ttf simhei.ttf` 

## 设置配置文件　matplotlibrc
修改配置文件　`/usr/local/lib/python2.7/dist-packages/matplotlib/mpl-data/matplotlibrc`, 取消注释
font.family, 和　font.sans-serif, 并在　font.sans-serif 后面添加　`Microsoft YaHei`

## 删除 matplotlib 字体缓存
python2 删除　`sudo rm ~/.cache/matplotlib/fontList.cache`，python3 删除　`sudo rm ~/.cache/matplotlib/fontList.py3k.cache`


