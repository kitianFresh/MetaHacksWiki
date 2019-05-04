---
title: "gopher-boxes"
date: 2018-04-21 18:55
---

[TOC]

# Go Tutorial & Material
## basics
 - [a tour of Go](https://tour.golang.org/)
 - [LEARNING GO](https://miek.nl/go/#preface)
 - [Go语言圣经](https://yar999.gitbooks.io/gopl-zh/content/ch0/ch0-01.html)

## go patterns
 - [Go design patterns example](https://github.com/tmrts/go-patterns)
 - [Evaluating the GO Programming Language with Design Patterns](https://ecs.victoria.ac.nz/foswiki/pub/Main/TechnicalReportSeries/ECSTR11-01.pdf)
## funny projects tutorial
 - [Building Blockchain in Go](https://jeiwan.cc/posts/building-blockchain-in-go-part-1/)
 - [Writing a Static Blog Generator in Go](https://zupzup.org/static-blog-generator-go/)



# Interative Go Programming
Jupyter 目前支持多种语言的内核，使得我们可以像学习Python这种动态的交互式解释型语言一样，学习GoLang这种静态的编译型语言，Amazing! 
目前github上star最多的是[Gophernotes](https://github.com/gopherdata/gophernotes), Mac OS X支持的不完美，不支持第三方库的动态导入，因此最好在Linux下使用，最简单的办法是直接使用Docker. 另一款是[lgo](https://github.com/yunabe/lgo#install).

采用Docker Jupyter Notebook，那关闭容器之后，当再次重启容器，就看不到jupyter 的服务端口以及token输出了，可以执行 `docker exec -it a7a0a00c50c9 jupyter notebook list` 查看当前运行的jupyter url。如果无法访问，可以执行 `jupyter notebook --ip 0.0.0.0 --no-browser --allow-root`

## 参考
 - [Announcing: a Golang kernel for Jupyter notebooks!](http://www.datadan.io/announcing-a-golang-kernel-for-jupyter-notebooks/)
 - [Interactive Go programming with Jupyter](https://medium.com/@yunabe/interactive-go-programming-with-jupyter-93fbf089aff1)




# Golang for Datascience
和 Python 数据科学生态numpy/pandas/matplotlib/sklearn 等对标的Go数据科学生态，Python 主要是易于学习上手，但是不适合生产环境开发和部署大规模高性能应用，而Golang这个Python和C语言的结合体，完美的实现了无锁的Concurrency和Parallelism，是开发分布式系统非常好的工程化语言，也被称为是21世纪的C语言，Golang的发明者同时也是C语言的发明者，Golang在做大规模数据处理和分布式系统非常有用，Golang在开发效率和执行编译效率得到权衡，非常适合开发部署和维护机器学习等工程应用，相信现在虽然是Python大行其道的时代，但是猜测随着机器学习和深度学习规模变大，应用场景和领域越来越多，越来越成熟的时候，可能在3-5年之后，Python就只适合做实验了，真正需要做的是快速做大规模数据清洗和预处理，快速训练模型，开发和部署哪些成熟的机器学习模型，Go语言会成为主流。

另外，Go语言会成为“上天入地”的最佳工程化语言，上天是指云计算，目前大部分云后端服务开始转向GO，比如Kubernetes和Docker技术，以及很多Web服务和微服务，入地是指边缘智能，也就是IOT，Go这种静态编译型语言在嵌入式设备当中应该比较方便，可以同时支持不同的架构和不同的平台交叉编译。

## 参考
 - [Golang libraries for data science](https://www.mjhall.org/golang-data-science-libraries/)
 - [Understanding Tensorflow using Go](https://pgaleone.eu/tensorflow/go/2017/05/29/understanding-tensorflow-using-go/)


# Go ORM

## 参考
-[Golang, ORMs, and why I am still not using one.](http://www.hydrogen18.com/blog/golang-orms-and-why-im-still-not-using-one.html)