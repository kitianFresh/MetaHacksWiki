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

