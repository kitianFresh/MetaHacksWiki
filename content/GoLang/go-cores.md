---
title: "go-cores"
date: 2018-04-22 18:06
---
[TOC]

# Go Features
## with
 - 并发
 - 垃圾回收
 - 高效的实现
 - 给人以动态类型体验的静态类型系统
 - 丰富但规模有限的标准库
 - 工具化
 - gofmt
 - 在大规模系统中的应用

## without
 - 没有隐式的数值转换
 - 没有构造函数和析构函数
 - 没有运算符重载
 - 没有默认默认参数
 - 没有继承
 - 没有泛型
 - 没有异常
 - 没有宏
 - 没有函数修饰
 - 没有线程局部存储

# Go Core Syntax
-[深度解密Go语言之关于 interface 的10个问题](https://gocn.vip/article/1717)

# Go Concurrency
## Communication by Channel without Locks
一个channel有发送和接受两个主要操作，都是通信行为。一个发送语句将一个值从一个goroutine通过channel发送到另一个执行接收操作的goroutine。
Channel还支持close操作，用于关闭channel，随后对基于该channel的任何发送操作都将导致panic异常。对一个已经被close过的channel之行接收操作依然可以接受到之前已经成功发送的数据；如果channel中已经没有数据的话讲产生一个零值的数据。

因此可以使用这样的单个channel当通知信号。关闭的时候就会告知接受者，自己已经关闭了，没有数据了。

## 无缓冲Channel
一个基于无缓存Channels的发送操作将导致发送者goroutine阻塞，直到另一个goroutine在相同的Channels上执行接收操作，当发送的值通过Channels成功传输之后，两个goroutine可以继续执行后面的语句。反之，如果接收操作先发生，那么接收者goroutine也将阻塞，直到有另一个goroutine在相同的Channels上执行发送操作。
基于无缓存Channels的发送和接收操作将导致两个goroutine做一次同步操作。因为这个原因，无缓存Channels有时候也被称为同步Channels。当通过一个无缓存Channels发送数据时，接收者收到数据发生在唤醒发送者goroutine之前



## Shared Memory and Locks
 - [Multiple Lock Based on Input in Golang](https://medium.com/@kf99916/multiple-lock-based-on-input-in-golang-74931a3c8230)

## Go coroutine so many and multi-core parallel programming
 - [Why you can have millions of Goroutines but only thousands of Java Threads](https://rcoh.me/posts/why-you-can-have-a-million-go-routines-but-only-1000-java-threads/)
 - [GoLang vs Python: deep dive into the concurrency](https://made2591.github.io/posts/go-py-benchmark)
 - [Multi-Core Parallel Programming in Go](http://www.ualr.edu/pxtang/papers/acc10.pdf)

# Go core concepts diff
 - [Go core diffs](https://www.godesignpatterns.com/)
