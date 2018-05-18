---
title: "networks-interview"
date: 2017-06-05 20:31
---

# tcp 三次握手和四次握手区别，为啥断开连接需要四次握手而不是三次？

 -[TCP3次握手连接协议和4次握手断开连接协议](http://blog.csdn.net/lostyears/article/details/7104349)

# 进程间通讯的各种方式？
 - 同一个主机内
  1. 进程间的简单控制和通讯，发送信号，信号机制
  2. 进程间的复杂控制和通讯，同步，使用信号量互斥锁
  3. 进程间需要发送大量数据的通讯
   - UNIX 管道，内存中实现的文件，匿名管道和有名管道FIFO．匿名管道只适用于有父子关系的进程之间，而有名管道适没有父子限制；但是管道只能发送字节形式的数据，需要自己解码和编码数据结构；
   - POSIX　或 System V 消息队列；可以定义消息的数据结构和格式；
   - 共享内存，内核申请一片内存，映射到相互通讯的进程空间地址中，以便进程间高效快速的共享数据, 由于多进程共享同一块内存，往往还需要使用进程间同步机制如信号量Semaphore；
   - 本地socket 套接字；
 - 不同主机之间
  1. 使用基于TCP/IP 协议栈的socket　套接字接口进行通讯; 可以利用已有协议或者实现自己的协议从而实现跨网络的进程间通信, 比如RPC调用，HTTP 连接，FTP 服务器等
  2. 基于底层协议如TCP/UDP/HTTP 实现的远程过程调用RPC/RMI等，他是通讯的更高级层次的抽象，让一个主机上的进程可以调用另外一个主机上进程的一个函数或 对象方法！其实就是通过双方协商协议和接口，然后通过网络、协议解析、压缩、同步/异步、并发等手段实现双方函数相互调用的过程。

# HTTP
## HTTP POST数据传输的四种方式
请求头中 Content-Type 的四种方式:
### application/x-www-form-urlencoded
最常见的普通的表单传输, 在原生 `<form>` 表单中, 如果不设置 `enctype` 属性, 最终就是  application/x-www-form-urlencoded 方式提交数据.

### multipart/form-data
该方式一般是传文件用的.

### application/json
传递 json 数据用. 现在的 Restful API 很多都使用 json 直接传输数据, 还有 SinglePage WebAPP

### text/xml
数据传输的另一种 xml. XML-RPC

## HTTP 传输大文件大图片的方法和优化思路

## cankao
- [HTTP-请求、响应、缓存](https://cnbin.github.io/blog/2016/02/20/http-qing-qiu-,-xiang-ying-,-huan-cun/)
- [HTTP协议知识扫盲](http://movesan.me/2017/03/06/http/)
- [四种常见的 POST 提交数据方式](https://imququ.com/post/four-ways-to-post-data-in-http.html)


# HTTPS
如何让HTTP变安全，HTTPS 的原理。

# RPC

## 安全的RPC
其实原理和HTTPS原理类似，在数据交换之前，引入密钥认证授权等手段进行安全认证秘钥交换等。可以完全把HTTPS 中间的基于TCP的TLS层拿过来使用。