---
title: "go-basics"
date: 2018-01-28 20:30
---
[TOC]

# Streaming IO in Go
[Streaming IO in Go](https://medium.com/learning-the-go-programming-language/streaming-io-in-go-d93507931185)

>bytes.NewReader creates a Reader from a byte slice (that is, a chunk of memory you already have in your program.) Useful if you want to pass a byte slice to some other API that expects a Reader. bufio.NewReader on the other hand is for wrapping existing Readers (usually ones whose Read method is relatively expensive, like a TCP connection or a file) to coalesce a bunch of easy-to-program small Read calls into a few larger ones.

>A bytes.Buffer is a buffer with two ends; you can only read from the 
start of it, and you can only write to the end of it. No seeking. 
There's no need to use both sides though; I often use it to construct 
a chunk of encoded stuff in memory (by passing it to serialization 
libraries expecting a Writer) then pull out the []byte at the end 
(with (*bytes.Buffer).Bytes()), then stop using the bytes.Buffer and 
pass the []byte I got to another layer for more work. 

A bytes.Reader is more explicitly only for turning a []byte into 
something that's a Reader; since no modifications are possible, it can 
also implement Seek and ReadAt without ambiguity, thus being more 
useful to pass to other functions expecting a ReaderAt or a ReadSeeker 
or whatever. 


# array/slice/map
## array
Go中的数组是支持直接拷贝赋值的，但是默认是一种浅拷贝行为。

C++ 需要通过重载运算符来进行拷贝赋值，重载运算符可以拷贝任意数据结构

Java其实是一切皆是对象，数组赋值其实只是对象赋值，因此默认只是拷贝了对象的引用，等同于没有数组拷贝赋值语句，数组拷贝需要自己实现或者通过标准库，Java是一种锤子思维模式(一切皆对象)，到处找钉子，不是钉子也要强制转成钉子。

## slice
需要区分底层数组的真实数据是否需要共享！！！


## map

# 基本数据结构对比
Go 中除了builtin 内建的 数组，切片，和词典，没有其他可动态扩容的数据结构。Go标准库提供了container包，但这个包也只是提供了 heap，list，ring三个数据结构。这主要原因就是Go的设计哲学决定的，Go只做low-system-level 的东西，和C语言的风格很像，只保留最简单的东西，同时又引入了一些高级语言比如Python的常用的数据结构，接口和多态等，可以快速上手，其他交给上层。所以，很多数据结构得自己实现一遍，可能比较蛋疼。

Java就非常丰富了，下面是Java的容器数据结构cheat-sheet, 这方面Java做得非常完善。Go内建的slice 和 map 也是最简单的基于数组和hash散列的设计，而Java则支持链表，红黑树，堆，并发等更复杂的设计，这些也是Java面试的经典考题。。。但是由于使用场景不同，可能Go还没有那么多需要设计复杂数据结构的场景（定位是基础设施和分布式系统），而Java的生态体系太庞大，囊括了J2EE 和大数据生态圈，所以支持这么多复杂的数据结构也是必然，大数据从hadoop开始已经发展了10多年了，发展出以Mesos和Yarn为首的资源管理，而MapReduce就只是算一个离线计算框架了，定位就和TensorFlow、Mxnet一样，这样就很容易部署在k8s上以容器化的形式运行了，资源管理都交给k8s，不知道以Kubernetes为首的基础设施生态体系，能不能在未来取代以hadoop为首的大数据基础设施，说实话，感觉非常难，历史包袱太重，但还是希望做下梦，万一实现了呢，据说twitter已经决定从Mesos迁移到Kubernetes上了。:-)

C++ 笔者已经很多年没碰过了，不敢评论，但是C++也是有丰富的容器数据结构支持。显然需要性能和速度的场景只能用C++，这是必然的，所有带GC的语言，不可能实时高性能，不然C++肯定没人用了，开发速度非常慢，学习曲线陡峭。。。。如果不需要实时高性能的场景，Go完全可以替代C++。但是这不可能，很多交易系统，路径规划，推荐系统，机器学习系统全部是C++写的，比如量化交易，你不能在交易的时候突然GC一下，结果错过了交易的最佳时间。。。比如机器学习里面到处都在操作矩阵和向量数据，此时还是C/C++操作矩阵最快速，通过汇编等硬件加速指令操作矩阵，因为最底层的语言往往最接近硬件表达能力的。

# 类型系统
Go不支持模板。。。。

## 内建类型

## 值接收者和指针接受者
```Go
// Add returns the time t+d.
func (t Time) Add(d Duration) Time {
	dsec := int64(d / 1e9)
	nsec := t.nsec() + int32(d%1e9)
	if nsec >= 1e9 {
		dsec++
		nsec -= 1e9
	} else if nsec < 0 {
		dsec--
		nsec += 1e9
	}
	t.wall = t.wall&^nsecMask | uint64(nsec) // update nsec
	t.addSec(dsec)
	if t.wall&hasMonotonic != 0 {
		te := t.ext + int64(d)
		if d < 0 && te > int64(t.ext) || d > 0 && te < int64(t.ext) {
			// Monotonic clock reading now out of range; degrade to wall-only.
			t.stripMono()
		} else {
			t.ext = te
		}
	}
	return t
}
```
是使用值接收者还是指针接收者，不应该由该方法是否修改了接收到的值来决定。这个决策 应该基于该类型的本质。这条规则的一个例外是，需要让类型值符合某个接口的时候，即便类型 的本质是非原始本质的，也可以选择使用值接收者声明方法

## 接口
 接口变量存储的类型
接口的变量里面可以存储任意类型的数值(该类型实现了某interface)。那么我们怎么反向知道这个变量里面实际保存了的是哪个类型的对象呢？目前常用的有两种方法：

comma-ok断言
value, ok = element.(T)，这里value就是变量的值，ok是一个bool类型，element是interface变量，T是断言的类型。如果element里面确实存储了T类型的数值，那么ok返回true，否则返回false。

switch测试
```Go
  switch value := element.(type) {
      case int:
          fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
      case string:
		   fmt.Printf("list[%d] is a string and its value is %s\n", index, value)
		   ...
```
element.(type)语法不能在switch外的任何逻辑里面使用，如果你要在switch外面判断一个类型就使用comma-ok。


# 代码开发构建测试和可读性
这方便Go完胜了Java/C++，Go 写起来还是非常快的。和C++比，不需要再手动释放内存了，和Java一定要OOP比，基于接口的设计真的是少了很多冗余代码。




# 引用完成在vscode已保存，引入的包就自动被删了；
如果是可导出的包，里面的函数必须要有大写开头的signature或者变量
