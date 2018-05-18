---
title: "pythonic"
date: 2017-04-23 22:19
---

[TOC]




# Python 字典排序
## python dict按照key 排序：
method 1.

```python
items = dict.items()
items.sort()
for key,value in items:
   print key, value # print key,dict[key]
```
2、method 2.

```python
print key, dict[key] for key in sorted(dict.keys())
```
　

## python dict按照value排序：
method 1：

把dictionary中的元素分离出来放到一个list中，对list排序，从而间接实现对dictionary的排序。这个“元素”可以是key，value或者item。

method2：

## 用lambda表达式来排序，更灵活：
```python
# sorted(iterable, cmp=None, key=None, reverse=False) --> new sorted list

sorted(dict.items(), lambda x, y: cmp(x[1], y[1]))
#降序
sorted(dict.items(), lambda x, y: cmp(x[1], y[1]), reverse=True)

print sorted(dict1.items(), key=lambda d: d[0])
print sorted(dict1.items(), key=lambda d: d[1])
```

