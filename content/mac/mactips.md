---
title: "MacTips"
date: 2017-07-29 18:33
---

[TOC]

# OSX 环境配置
OS X 下载 brew
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" < /dev/null 2> /dev/null ; brew install caskroom/cask/brew-cask 2> /dev/null

brew cask install pycharm

## Mac 快捷键
Mac 下的 home 和 end
Ctrl+A：到行首（达到Home键的效果）
Ctrl+E：到行尾（达到End键的效果）
Ctrl+N：到下一行
Ctrl+P：到上一行
Ctrl+K：从光标处开始删除，知道行尾

 - [A Quick and Easy Guide to tmux](http://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)
 - [Tmux 简介与使用](http://kuanghy.github.io/2016/09/29/tmux)


# Mac OS X新版本系统root用户也无法安装python包
新系统引入了保护机制，`sudo pip install xxx` 也无法安装， 或出现权限错误，`OSError: [Errno 1] Operation not permitted:'/System/Library/Frameworks/Python.framework/Versions/2.7/share'`由于El Capitan引入了SIP机制(System Integrity Protection)，默认下系统启用SIP系统完整性保护机制，无论是对于硬盘还是运行时的进程限制对系统目录的写操作。
现在的解决办法是取消SIP机制，具体做法是：

重启电脑，按住Command+R(直到出现苹果标志)进入Recovery Mode(恢复模式)
左上角菜单里找到实用工具 -> 终端
输入csrutil disable回车
重启Mac即可
如果想重新启动SIP机制重复上述步骤改用csrutil enable即可

另外， 新版本的 ipython 也会出问题， 最好安装 jupyter. `sudo pip install jupyter`


## 在MAC OS 上安装MYSQL
Download MySQL for OS X
Download latest stable version of MySQL server for your OS X version and architecture. Link: http://dev.mysql.com/downloads/mysql/. Please make sure you download the .dmg file.

Unpack download .dmg file
Click on the downloaded .dmg file and unpack it. Click on the mysql server package from unpacked files.

Install MySQL from downloaded file
Install MySQL server by clicking on the mysql package to open up the installer. If you want to install the startup script to automatically start MySQL server at the time of system startup, you should also install the start up package of mysql now (included in the unpacked MySQL package that you just downloaded and unpacked as a separate file).

Once the installer has finished successfully, all MySQL related files should be installed under /usr/local/mysql-VERSION directory. A link /usr/local/mysql (pointing to the MySQL installation directory) should also have been created for your convenience.

Install and setup auto start package for MySQL on OS X
If you had installed the automatic startup package at the time of installation, you should now be able to start MySQL running the following command in OS X Terminal window or by restarting the operating system:

$ sudo /Library/StartupItems/MySQLCOM/MySQLCOM start
As soon as you run the command above, your system might ask for permission to allow MySQL server to accept incoming connections. You must give the permission to listen on port 3306 (or change it later to run on a different port).

In case, you did not install the startup script, you have to run mysqld_safe under /usr/local/mysql/bin/mysqld_safe script.

If you want to disable automatic MySQL starts at the time of system startup, you can change the value of MYSQLCOM in /etc/hostconfig to "-NO-" (without quotes).

You should also consider including /usr/local/mysql/bin/ in your system PATH variable.

Connect to installed MySQL server
In order to connect to MySQL using command line client, do the following from OSX terminal:


通过命令行登陆看是否安装 `$ /usr/local/mysql/bin/mysql -uroot`
最后配置环境变量
```sh
# for mysql #
export MYSQL_HOME=/usr/local/mysql
export PATH=$PATH:$MYSQL_HOME/bin
# mysql 服务启动和停止命令
alias mysqlstart='sudo /usr/local/mysql/support-files/mysql.server start'
alias mysqlstop='sudo /usr/local/mysql/support-files/mysql.server stop'
```

## 重置MySQL root 密码
1. 停止MySQL服务 `sudo service mysql stop`

2. 设置MySQL安全模式 `sudo mysqld_safe --skip-grant-tables --skip-networking &`

3. 无密码进入MySQL `mysql -u root`

4. 重置密码
```sql
mysql> use mysql;  
mysql> update user set password=PASSWORD("mynewpassword") where User='root';  
mysql> flush privileges; 
```
如果 update 不成功，比如报错说没有 password 字段，看看自己的MYSQL版本，新版本 5.7 密码字段叫 `authentication_string`
```sql
UPDATE user SET authentication_string=PASSWORD("NEWPASSWORD") WHERE User='root';
```
5. 重启MySQL服务 `sudo service mysql restart`

6. 苹果上重启后可能还遇到问题`ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.`
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'YourNewPass';
```

## Mac 配置科大VPN教程
[科大VPN教程](http://netfee.ustc.edu.cn/faq/index.html#howtosetupvpn)

# iTerm2 和 Tmux以及Screen配置
## iTerm2
[iTerm2](https://www.iterm2.com/) 直接到官网下载并安装即可. 这里说一下配置iTerm2主题颜色以及背景透明。让你的shell色彩斑斓，提高文字区分度。
[iTerm2-Color-Schemes](https://github.com/mbadolato/iTerm2-Color-Schemes) 里面有很多款不同的主题，按照教程配置主题即可。背景色透明直接在 iTerm2-->Preferences-->Profiles-->Window-->Transparency中自行调节。
最后，还需要将下面的脚本粘贴到 `~/.bash_profile` 配置文件中，并执行 `source ~/.bash_profile` 使其生效即可。主要是配置`ls` 命令的显示。

command+alt+t 即可唤出 ITerm2 终端。
```bash
export CLICOLOR=1
export LSCOLORS=gxfxcxdxbxegedabagacad
export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;36m\]\w\[\033[00m\]\$ '
export TERM=xterm-color
```

## screen 使用简介
screen 和 tmux有类似的服务。只是screen开启会话后不会新建窗口，还是在当前窗口。

1. screen 创建一个session
```
screen -S myNewSession
或者
screen 创建一个名字默认是ttys[ID] 的session.
```
2. 从当前 session 暂时断开退出。 `ctrl+a+d`

3. 列出当前所有的session. `screen -ls` 
```
There are screens on:
	28007.ttys005.tianqideMacBook-Pro	(Detached)
	28091.myNewSession	(Detached)
2 Sockets in /var/folders/bb/ywfc1vbs5lx_blqpl5ssrvv40000gp/T/.screen.
```
4. 恢复某一个session。`screen -r myNewSession`

5. 强制杀死一个 session. `kill -9 28091`
```
There are screens on:
	28007.ttys005.tianqideMacBook-Pro	(Detached)
	28091.myNewSession	(Dead ???)
Remove dead screens with 'screen -wipe'.
2 Sockets in /var/folders/bb/ywfc1vbs5lx_blqpl5ssrvv40000gp/T/.screen.
```
6. 按照提示清理。 `screen -wipe`

7. `Cannot open your terminal '/dev/pts/0' - please check.`
1. Sign out and properly connect / sign in as the user you wish to use.
2. Run `script /dev/null` to own the shell (more info over at Server Fault); then try screen again.


## 配置 tmux

Tmux 是 Terminal multiplexer 的缩写， 其实就是 终端可以复用的意思。 

场景就是，第一种是在本地电脑，你打开了好几个terminal， 每一个terminal由当前的工作环境， 然后这个时候没电了或者你电脑关机了，那么这些terminal的工作状态就没有了。如果使用tmux记录下来，那么这些terminal下次还可以继续使用，并且恢复到关闭时的状态。

第二种就是 ssh 登陆到远程主机， 然后ssh 基本上很容易断开，一般使用 nohup & 在后台运行命令， 但是如果使用 tmux， 就可以记录下来，关闭了也没关系，还可以再次恢复。

执行 tmux -CC 之后会开启一个 session， tmux -CC 默认会以数字编号命名一个回话session， 并开启一个新的iterm窗口，在这个窗口即回话下面， 使用 command+T 可以创建多个窗口（注意不是command + D, 也不是 command + N）， 原来 tmux -CC 创建的窗口会阻塞，可以通过 esc 退出， 然后再通过 tmux -CC attach 恢复， 但是 tmux attach 恢复出来是窗口叠加在一起的， 加 CC 参才会原样恢复。

另外， tmux new -s session\_name, 可以给一个session起名字， 以后可以使用 tmux attach -t session\_name 恢复某个会话。


## MAC 安装 lightgbm 出现问题
可能是mac gcc 的原因，最好在编译的时候指定编译器, `-DCMAKE_C_COMPILER=/usr/local/Cellar/gcc/8.1.0/bin/gcc-8 -DCMAKE_CXX_COMPILER=/usr/local/Cellar/gcc/8.1.0/bin/g++-8`

```sh
brew install cmake  
brew install gcc --without-multilib  
git clone --recursive https://github.com/Microsoft/LightGBM ; cd LightGBM  
mkdir build ; cd build  
cmake -DCMAKE_C_COMPILER=/usr/local/Cellar/gcc/8.1.0/bin/gcc-8 -DCMAKE_CXX_COMPILER=/usr/local/Cellar/gcc/8.1.0/bin/g++-8 ..
make -j  
```