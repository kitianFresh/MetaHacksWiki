---
title: "package-install"
date: 2017-04-23 21:24
---

## 安装图像处理模块

```
sudo pip install Pillow
//tesseract-ocr库
sudo apt-get install tesseract-ocr
// tesseract-ocr pthon wrapper
sudo pip install pytesseract

//tesseract-ocr默认没有中文字符集，下载中文字符集
wget https://github.com/tesseract-ocr/tessdata/raw/master/chi_sim.traineddata

//然后copy到目录/usr/share/tesseract-ocr/tessdata/
sudo cp chi_sim.traineddata /usr/share/tesseract-ocr/tessdata/

//example
from PIL import Image
import pytesseract
print(pytesseract.image_to_string(Image.open('血常规.jpg'), lang='chi_sim'))

```
## 安装Python机器视觉编程[英文](http://programmingcomputervision.com/)[中文](http://yongyuan.name/pcvwithpython/)中PCV库
依赖
```
sudo pip install numpy
sudo pip install matplotlib
sudo pip install scipy

```
[下载库到本地](https://github.com/jesolem/PCV/zipball/master)
```
cd jesolem-PCV-376d597/
sudo python setup.py install

//以下命令可以用来确定Ubuntu系统拥有的字体和位置
fc-list :lang=zh 

import PCV
```

## 参考
 - [基本的图像处理与 OCR 文字识别工具总结 (Python)](https://testerhome.com/topics/4615)


# 低配机器安装tensorflow出现MemoryError
原因是pip 安装默认会进行缓存，全部读到内存中，过大的包就会出现该错误
操作系统    Ubuntu Server 16.04.1 LTS 64位
CPU 1核
内存  1GB
系统盘 20GB(云硬盘)   disk-qy4kp18l
公网带宽    1Mbps
```
sudo pip  --no-cache-dir install --upgrade https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.12.0rc0-cp27-none-linux_x86_64.whl
```
以下两者的区别，在翻墙的时候表现的就尤其明显，前者会非常慢，这里就是因为 后者 是package 安装形式，主要是从Ubuntu的软件源找包并安装，国内都会有相应的镜像，速度不慢，但是前者就会直接到Python的package站点下载，非常慢，基本下不动
```
sudo pip install numpy
sudo apt-get install python-numpy
```
基本上安装Python包都可以使用 sudo apt-get install python-XXX 或者直接 sudo pip install XXX，如果前者找不到，看看提示，是不是名字不一样，基本大多数包都支持 Ubuntu package



# MySQL-python 安装
centos 上安装的时候报错 `EnvironmentError: mysql_config not found`； 网上说 装 mysql-devel， 但是因为安装的 mysql 是 percona 因此装不了， 正确的安装姿势是

查看 `yum list installed | grep -i percona` 是否安装了 percona ，然后再 `yum whatprovides mysql_config` 或 `yum whatprovides */mysql_config` 列表中找到 devel 字样的开发包。
安装 Percona-Server-devel-55-5.5.34-rel32.0.591.rhel6.x86_64


# pycurl 失败
`    __main__.ConfigurationError: Could not run curl-config: [Errno 2] No such file or directory`
sudo yum install libcurl-openssl-devel

# pip 安装走国内镜像源的方法
pip 安装python包其实可以添加参数 `pip install --index-url  http://pypi.douban.com/simple`， 但是太麻烦，直接在$HOME 目录下新建 .pip 目录，在此目录下新建 pip.conf 文件, 加入如下几句:
```python
[global]
index-url = http://pypi.douban.com/simple
trusted-host = pypi.douban.com
```
或者是直接使用 apt-get 或者 yum 来安装 python 包；

# 安装pyenv
首先更新，必须的，不管安装什么， 要不然会出错。`yum update (apt-get update)`。 然后使用 `curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash` 安装 pyenv;

`pyenv versions` 查看版本，`pyenv global 2.7.13` 切换版本， 没有的话，需要先安装， `pyenv install 2.7.13` 可以安装各种版本以及Anaconda。

但是一般你会安装的很慢，下载 python 安装包很慢。可以修改对应版本下的下载url。 在 `.pyenv/plugins/python-build/share/python-build/` 目录下修改对应安装版本的构建文件， 例如 2.7.13 就是 `.pyenv/plugins/python-build/share/python-build/2.7.13`；打开后编辑 url， 将原来的换成自己的一个可以很快下载的站点。
```python
#require_gcc
install_package "openssl-1.0.2k" "https://www.openssl.org/source/openssl-1.0.2k.tar.gz#6b3977c61f2aedf0f96367dcfb5c6e578cf37e7b8d913b4ecb6643c3cb88d8c0" mac_openssl --if has_broken_mac_openssl
install_package "readline-6.3" "https://ftpmirror.gnu.org/readline/readline-6.3.tar.gz#56ba6071b9462f980c5a72ab0023893b65ba6debb4eeb475d7a563dc65cafd43" standard --if has_broken_mac_readline
#if has_tar_xz_support; then
#  install_package "Python-2.7.13" "https://www.python.org/ftp/python/2.7.13/Python-2.7.13.tar.xz#35d543986882f78261f97787fd3e06274bfa6df29fac9b4a94f73930ff98f731" ldflags_dirs standard verify_py27 ensurepip
#else
#  install_package "Python-2.7.13" "https://www.python.org/ftp/python/2.7.13/Python-2.7.13.tgz#a4f05a0720ce0fd92626f0278b6b433eee9a6173ddf2bced7957dfb599a5ece1" ldflags_dirs standard verify_py27 ensurepip
#fi

install_package "Python-2.7.13" "http://172.18.177.215:8000/Python-2.7.13.tar.xz" ldflags_dirs standard verify_py27 ensurepip
```
