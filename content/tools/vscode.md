---
title: "vscode"
date: 2017-04-23 21:25
---

# 安装VScode
1.下载[压缩包](https://code.visualstudio.com/Download)code-stable-code_1.7.2-1479766213_amd64.tar.gz
```
cd Download

//解压到/opt目录
sudo tar -xzf code-stable-code_1.7.2-1479766213_amd64.tar.gz -C /opt

//创建VSCode软连接，目的是方便以后更换新版本VSCode就不用配置了，直接替换当前的VSCode-linux-x64
sudo ln -s /opt/VSCode-linux-x64/ /opt/VSCode

//创建运行的软连接，这样就可以在任意目录运行code,其实可以直接进入VSCode-linux-x64目录下面运行code，
//或者直接配置环境变量，以下配置是为了保持系统环境变量的干净整洁
//这里因为/usr/local/bin默认总是在环境变量中，所以下载的软件都可以这样做，
sudo ln -s /opt/VSCode/code /usr/local/bin/code

code .
```
2.可以配置桌面快捷方式，编辑文件： sudo vi /usr/share/applications/VSCode.desktop
```
#!/usr/bin/env xdg-open

[Desktop Entry]
Version=1.7.2
Type=Application
Terminal=false
Exec=/opt/VSCode/code
Name=VSCode
Icon=/opt/VSCode/resources/app/resources/linux/code.png
Categories=Development
```