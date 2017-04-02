---
title: "ConcurrencyModel"
date: 2017-03-28 22:33
---

# 并行计算
两大类并行

## 多进程或多线程并行
race condition dead lock

## 流水线并行
non-blocking io event-driven system

## 函数式并行(Functional Parallelism)
单个任务可以被拆分成多个部分来处理，每个部分都是一个函数，数据经过这些函数的时候是并行处理的，其实就是流水线
mapreduce; 但是对于本来就运行多个不同类型的任务的系统，比如 web server database server 都是不适合用函数式并行的，因为cpu本来就在处理多任务