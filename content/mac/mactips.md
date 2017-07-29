---
title: "MacTips"
date: 2017-07-29 18:33
---

# OSX 环境配置
OS X 下载 brew
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" < /dev/null 2> /dev/null ; brew install caskroom/cask/brew-cask 2> /dev/null

brew cask install pycharm

# Mac 快捷键
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