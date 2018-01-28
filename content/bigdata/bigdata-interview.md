---
title: "bigdata-interview.md"
date: 2017-05-19 21:12
---


# hive sql　的执行流程

# java 中抽象类和接口的区别
类如果要实现一个接口，它必须要实现接口声明的所有方法。但是，类可以不实现抽象类声明的所有方法，当然，在这种情况下，类也必须得声明成是抽象的。抽象类可以在不提供接口方法实现的情况下实现接口。Java接口中声明的变量默认都是final的。抽象类可以包含非final的变量。Java接口中的成员函数默认是public的。抽象类的成员函数可以是private，protected或者是public。接口是绝对抽象的，不可以被实例化。抽象类也不可以被实例化，但是，如果它包含main方法的话是可以被调用的。

-[参考]https://www.zhihu.com/question/20149818/answer/97282207）


# hadoop mapreduce 如何解决数据倾斜



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