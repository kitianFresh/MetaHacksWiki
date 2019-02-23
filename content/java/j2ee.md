---
title: "j2ee"
date: 2018-11-29 18:38
---

# Install JRE & JDK
## Ubuntu16.04
```
sudo apt-get update
sudo apt-get install default-jre
sudo apt-get install default-jdk
```

# Install Tomcat 9
```
wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.13/bin/apache-tomcat-9.0.13.tar.gz

sudo tar xvf apache-tomcat-9.0.13.tar.gz -C /opt

sudo ln -s apache-tomcat-9.0.13 tomcat9
```
新建服务单元 sudo vim /etc/systemd/system/tomcat9.service
```
[Unit]
Description=Tomcat9
After=network.target

[Service]
Type=forking
User=root
Group=root

Environment=CATALINA_PID=/opt/tomcat9/temp/tomcat9.pid
Environment=JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
Environment=CATALINA_HOME=/opt/tomcat9
Environment=CATALINA_BASE=/opt/tomcat9
Environment="CATALINA_OPTS=-Xms512m -Xmx512m"
Environment="JAVA_OPTS=-Dfile.encoding=UTF-8 -Dnet.sf.ehcache.skipUpdateCheck=true -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+UseParNewGC"

ExecStart=/opt/tomcat9/bin/startup.sh
ExecStop=/opt/tomcat9/bin/shutdown.sh

[Install]
WantedBy=multi-user.target

```

启动tomcat服务
```
sudo systemctl daemon-reload
sudo systemctl start tomcat9
sudo systemctl enable tomcat9
```

## firewall
设置开放8080端口，然后访问 localhost:8080
```
sudo ufw allow 8080
```

- [How To Install Apache Tomcat 9 on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/install-tomcat-9-ubuntu-1804)


# Install MySQL
## 1. install
密码设置成1234
```
sudo apt-get update
sudo apt-get install mysql-server
```
## 2. verify
```
// Show packages relating to mysql
$ dpkg --get-selections | grep mysql
mysql-client-5.7       install
mysql-client-core-5.7  install
mysql-common           install
mysql-server           install
mysql-server-5.7       install
mysql-server-core-5.7  install


// Check the details of a package
$ dpkg --status mysql-server
......
Version: 5.7.16-0ubuntu0.16.04.1
.......

// List the installed files of a package
$ dpkg --listfiles mysql-server
.......

// Check the location of "mysqld" (MySQL server deamon)
$ which mysqld
/usr/sbin/mysqld
$ whereis mysqld
mysqld: /usr/sbin/mysqld /usr/share/man/man8/mysqld.8.gz
$ man mysqld   // Read the manual

// Check the location of "mysql" (MySQL command-line client)
$ which mysql
/usr/bin/mysql
$ whereis mysql
mysql: /usr/bin/mysql /etc/mysql /usr/lib/mysql /usr/bin/X11/mysql /usr/share/mysql /usr/share/man/man1/mysql.1.gz
$ man mysql   // Read the manual
```

## 3. config mysql server（可乎略）
Step 3: Configure MySQL Server
MySQL reads the startup options from the files shown below, in the specified order (top files are read first, files read later take precedence) (Reference: http://dev.mysql.com/doc/refman/5.7/en/option-files.html)
```
/etc/my.cnf
/etc/mysql/my.cnf
SYSCONFDIR/my.cnf
$MYSQL_HOME/my.cnf (server only)
The file specified in --defaults-extra-file startup option, if any
~/.my.cnf
~/.mylogin.cnf (client only)
```
The installation default /etc/mysql/my.cnf includes directories /etc/mysql/conf.d/ and /etc/mysql/mysql.conf.d/. The /etc/mysql/conf.d/mysql.cnf is empty. Hence, the main configuaration file is /etc/mysql/mysql.conf.d/mysqld.cnf.

Browse through /etc/mysql/mysql.conf.d/mysqld.cnf:
```
[mysqld]
user            = mysql
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
......
log_error = /var/log/mysql/error.log
......
```
- A special user called "mysql" is created to run the MySQL server.
- The server runs on the default port number of 3306.
- The data directory is located at /var/lib/mysql (owned by mysql:mysql).
- The error log is located at /var/log/mysql/error.log.


## 4. start/shutdown mysql server
MySQL is run as a service called "mysql" (configured at "/etc/init.d/mysql"), which is started automatically after boot. To start/stop/restart mysql, you could:
```
$ sudo service mysql start
$ sudo service mysql stop
$ sudo service mysql restart
   // Stop and start
$ sudo service mysql status
   // Show the status
```

# jdbc
安装 `sudo apt-get install libmysql-java`; 然后 `ll /usr/share/java/mysql-connector-java*` 查看 `mysql-connector-java.jar` 的位置。拷贝到`/usr/lib/jvm/java-1.8.0-openjdk-amd64/jre/lib/ext`目录下，这样webapps就不需要这个包了。另一种方法是直接拷贝到`/opt/tomcat9/webapps/Yelper/WEB-INF/lib` 目录下面也可以。


# 应用部署
将应用放入 `/opt/tomcat9/webapps` 目录下即可。然后访问 `localhost:8080/Yelper`
