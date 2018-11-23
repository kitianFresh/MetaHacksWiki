---
title: "rabbitmq"
date: 2017-09-06 20:12
---

# install on centos
```bash
# Add and enable relevant application repositories:
# Note: We are also enabling third party remi package repositories.
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
sudo rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm
# Finally, download and install Erlang:
sudo yum install -y erlang
Once we have Erlang, we can continue with installing RabbitMQ:
# Download the latest RabbitMQ package using wget:
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.2.2/rabbitmq-server-3.2.2-1.noarch.rpm
# Add the necessary keys for verification:
sudo rpm --import http://www.rabbitmq.com/rabbitmq-signing-key-public.asc
# Install the .RPM package using YUM:
sudo yum install rabbitmq-server-3.2.2-1.noarch.rpm
```

# manage on centos
配置开机启动
```
chkconfig rabbitmq-server on
```
开启/关闭/重启/状态 操作
```sh
/sbin/service rabbitmq-server start
/sbin/service rabbitmq-server stop
/sbin/service rabbitmq-server restart
/sbin/service rabbitmq-server status
```


# install on ubuntu
```
apt-get    update 
apt-get -y upgrade

echo "deb http://www.rabbitmq.com/debian/ testing main" >> /etc/apt/sources.list
curl http://www.rabbitmq.com/rabbitmq-signing-key-public.asc | sudo apt-key add -

apt-get update
sudo apt-get install rabbitmq-server
sudo nano /etc/default/rabbitmq-server
```
# manage on ubuntu
```bash
service rabbitmq-server start
service rabbitmq-server stop
service rabbitmq-server restart
service rabbitmq-server status
rabbitmqctl status
```


# install rabbitmq on mac
使用 brew 安装
```
brew install rabbitmq
```
环境变量, brew 安装的 rabbitmq 可执行文件在 /usr/local/sbin 里，实际上都是软连接，真实目录一般在 `/usr/local/Cellar/rabbitmq/3.6.11/sbin`
```
export PATH=$PATH:/usr/local/sbin
```
rabbitmqctl status

# config 
目录情况, 一般 rabbitmq.config 和 rabbitmq-env.conf都在
```
Generic UNIX - $RABBITMQ_HOME/etc/rabbitmq/
Debian - /etc/rabbitmq/
RPM - /etc/rabbitmq/
Mac OS X (Homebrew) - ${install_prefix}/etc/rabbitmq/, the Homebrew prefix is usually /usr/local
Windows - %APPDATA%\RabbitMQ\
```

# 多语言客户端
## python 
sudo pip install pika