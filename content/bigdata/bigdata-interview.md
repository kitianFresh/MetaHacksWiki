---
title: "bigdata-interview.md"
date: 2017-05-19 21:12
---
[TOC]

# hive sql　的执行流程

 - [Hive SQL的编译过程](https://tech.meituan.com/hive-sql-to-mapreduce.html)

# java 中抽象类和接口的区别
类如果要实现一个接口，它必须要实现接口声明的所有方法。但是，类可以不实现抽象类声明的所有方法，当然，在这种情况下，类也必须得声明成是抽象的。抽象类可以在不提供接口方法实现的情况下实现接口。Java接口中声明的变量默认都是final的。抽象类可以包含非final的变量。Java接口中的成员函数默认是public的。抽象类的成员函数可以是private，protected或者是public。接口是绝对抽象的，不可以被实例化。抽象类也不可以被实例化，但是，如果它包含main方法的话是可以被调用的。

 - [参考](https://www.zhihu.com/question/20149818/answer/97282207)


# hadoop 如何合理设置块大小，块大小为何一般是　64M 128M
## 设置过小
   - 一块过小，块数就变多，磁盘文件都是按页存取的，这样磁盘寻道时间就变多了，性能降低
   - 块儿数变多，元数据就变多，由于 HDFS 元数据全部存在 Master 的内存之中，因此会导致内存消耗过大

## 设置过大
 - JVM 内存有限制，太大的块必然导致读取速度变慢，而且可能内存不够
 - Map 约束。快太大的话，Map 端归并排序成一块会变慢

[Block块大小设置](http://www.cnblogs.com/Dhouse/p/6901028.html)


# 如何设置 mapreduce 的　Reducer 个数，　Mapper 个数呢？
Reducer 的个数我们可以直接在程序中设定，job.setNumReduceTasks(0);　但是对于 Mapper 好像是系统自动调配的，那么如何设置 Mapper 个数呢？　

Mapper 个数其实和输入数据文件大小，块大小，自定义的输入处理器都有关系.

Mapper 的输入是 split, 而splitSize 由文件大小，个数以及块大小和配置参数决定的。小文件可能会被合并，大文件会被拆分；因此需要配置好这些参数。

```java
 mapreduce.input.fileinputformat.split.minsize //启动map最小的split size大小，默认0 小于这个值会合并
mapreduce.input.fileinputformat.split.maxsize //启动map最大的split size大小，默认256M 大于这个值会切分
dfs.block.size//block块大小，默认64M
计算公式：splitSize =  Math.max(minSize, Math.min(maxSize, blockSize)
```

## 减少 Mapper 的方法
1. 输入文件size巨大，但不是小文件; 增大　minSize
2. 输入文件数量巨大，但是都是小文件；合并小文件


[MapReduce原理和优化](http://blog.csdn.net/aijiudu/article/details/72353510)



# hadoop/hive mapreduce 如何解决数据倾斜？
## join的key值发生倾斜，key值包含很多空值或是异常值
这种情况可以对异常值赋一个随机值来分散key

## key 是有效值
### 增加reduce 个数
### combine 进行map端连接

### 抽样和范围分区
比如 TotalOrderPartitioner;可以通过对原始数据进行抽样得到的结果集来预设分区边界值。TotalOrderPartitioner中的范围分区器可以通过预设的分区边界值进行分区。因此它也可以很好地用在矫正数据中的部分键的数据倾斜问题。

### 自定义分区
将某些特殊的出现多次的　key 单独分到某些特定的 reducer

### 小表与大表关联

此时，可以通过mapjoin来优化，
```
set hive.auto.convert.join = true ; //将小表刷入内存中  

set hive.mapjoin.smalltable.filesize = 2500000 ;//刷入内存表的大小(字节)  
```
 - [Hadoop中MapReduce中解决数据倾斜的方法](https://ych0112xzz.github.io/2017/02/02/MapReducece-Skew/)
 - [Handling Data Skew in MapReduce Cluster by Using Partition Tuning](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5415866/)
 - [Join Type in Hive Skewed Join](https://weidongzhou.wordpress.com/2017/06/08/join-type-in-hive-skewed-join/)
 - [减小数据倾斜的性能损失](https://www.cnblogs.com/datacloud/p/3601624.html)

# 大数据统计　topk 不同场景的不同方案　时间复杂度不能超过 O(nlgn)
1. 排好序的topk　或者　不用排好序
2. 完全在内存中进行　或者　内存装不下

## 1. 完全在内存中进行且排好序的topK
非常常见的就是在内存中定义一个容量为　K 的小根堆。并输入k个元素建立该堆，然后一个个输入剩余n-K　个元素进行堆调整，如果输入元素比堆顶元素小，必定不是Topk中的元素，丢弃，否则删除堆顶元素，插入该元素调整堆。最后结果就是一个有序的TopK;　nlogK
或者直接在内存中进行堆排序，快速排序等。

## 2. 完全在内存中不一定要排好序的TopK
这个就可以直接使用快速排序的变种，因为快速排序每进行一轮排序，就会定位一个元素的最终序位rank.这里采用递减的快速排序 如果　K > rank, 说明A[left, rank]一定是解的一部分，剩余　(K - (rank-left+1)) 需要在　A[rank+1...right] 中寻找，问题缩小为　A[rank+1...right] 的Top　(K - (rank-left+1))问题，如果 K < rank, 问题缩小为　A[left...rank-1] 的　Top K 问题； logn

## 3. 内存一次装不下且需要排好序
则还是在内存中维护一个堆, 方式和１一样，只是从外存分批读取剩余数据

## 4. 内存一次转不下不一定需要排序
可以对文件进行切分成　K 份，每一份能装入内存，那么可以求每一份的　topk, 然后再从ｋ 份　topk 中求出 最终的 topk
如果每一份还是不能装入内存，可以使用优化，分布式并行计算。每一份分发到不同的主机，每一个主机内并行的进行　TopK 的求解，　最后汇总结果，其实就是　Map Reduce



# 数据划分并采用堆进行排序

```python
import os
import os.path
import operator
import heapq

"""
sort users' queries by frequency
1. hashing queries and dividing into 10 files. (hash(query)%10)
2. counting the number queries and sorting in each file using hashtable.
3. merging files using heap queue algorithm.
"""

datadir  = "d:/querysort/data/"
tempdir  = "d:/querysort/temp/"
destfile = "d:/querysort/sorted.txt"

def hashfiles():
    fs = []
    if not os.path.exists(tempdir):
        os.makedirs(tempdir)
    for f in range(0,10):
        fs.append(open(tempdir + str(f), 'w'))
    
    for parent, dirnames, filenames in os.walk(datadir):
        for filename in filenames:
            f = open(os.path.join(parent, filename),'r')
            for query in f:
                fs[hash(query)%10].write(query)
            f.close()          

    for f in fs:
         f.close()
     
                
def sortqueryinfile():
    fs = []
    if not os.path.exists(tempdir):
        return
    for f in range(0,10):
        fs.append(open(tempdir + str(f), 'r+'))

    for f in fs:
        D = {}
        for query in f:
            if query in D:
                D[query] += 1
            else:
                D[query] = 1
        sorted_D = sorted(D.iteritems(), key=operator.itemgetter(1), reverse=True)
        f.seek(0,0)
        f.truncate()
        for item in sorted_D:
            f.write(str(item[1]) + "\t" + item[0])
        f.close()

def decorated_file(f):
    """ Yields an easily sortable tuple. 
    """
    for line in f:
        count, query = line.split('\t',2)
        yield (-int(count), query)

def mergefiles():
    fs = []
    if not os.path.exists(tempdir):
        return
    for f in range(0,10):
        fs.append(open(tempdir + str(f), 'r+'))
    f_dest = open(destfile,"w")
    lines_written = 0
    for line in heapq.merge(*[decorated_file(f) for f in fs]):
        f_dest.write(line[1])
        lines_written += 1
    return lines_written

     
if __name__ == '__main__':
    hashfiles()
    sortqueryinfile()
    print "sorting completed, total queries: ", mergefiles()
```