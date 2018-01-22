---
title: "hive"
date: 2018-01-21 17:15
---

# Hive 排序
## sort by
局部排序，每一个 reducer 结果之内是有序的, 但是多个 reducer 之间是无序的；
```sql
#set the number of reducers to 2
0: jdbc:hive2://localhost:10000> set mapred.reduce.tasks=2;

0: jdbc:hive2://localhost:10000> select * from employee sort by salary;
+--------------+----------------+---------------+------------------+----------------------+--+
| employee.id  | employee.name  | employee.age  | employee.salary  | employee.department  |
+--------------+----------------+---------------+------------------+----------------------+--+
| 10001        | rajesh         | 29            | 50000            | BIGDATA              |
| 1            | aarti          | 28            | 55000            | HR                   |
| 2            | sakshi         | 22            | 60000            | HR                   |
| 10003        | dinesh         | 35            | 70000            | BIGDATA              |
| 3            | mahesh         | 25            | 25000            | HR                   |
| 10002        | rahul          | 23            | 250000           | BIGDATA              |
+--------------+----------------+---------------+------------------+----------------------+--+
```
## order by
全局排序，每一个 reducer 结果之内是有序的额, reducer 与 reducer 之间是有序的，结果可以直接拼接在一起构成全序；
```sql
#set the number of reducers to 2
0: jdbc:hive2://localhost:10000> set mapred.reduce.tasks=2;

0: jdbc:hive2://localhost:10000> select * from employee order by salary; 
+--------------+----------------+---------------+------------------+----------------------+--+
| employee.id  | employee.name  | employee.age  | employee.salary  | employee.department  |
+--------------+----------------+---------------+------------------+----------------------+--+
| 3            | mahesh         | 25            | 25000            | HR                   |
| 10001        | rajesh         | 29            | 50000            | BIGDATA              |
| 1            | aarti          | 28            | 55000            | HR                   |
| 2            | sakshi         | 22            | 60000            | HR                   |
| 10003        | dinesh         | 35            | 70000            | BIGDATA              |
| 10002        | rahul          | 23            | 250000           | BIGDATA              |
+--------------+----------------+---------------+------------------+----------------------+--+
```
## distribute by
组间排序，每一个 reducer 结果之内无序， reducer 与 reducer 之间有序；
```sql
#set the number of reducers to 2
0: jdbc:hive2://localhost:10000> set mapred.reduce.tasks=2;

0: jdbc:hive2://localhost:10000> select * from employee distribute by id;
+--------------+----------------+---------------+------------------+----------------------+--+
| employee.id  | employee.name  | employee.age  | employee.salary  | employee.department  |
+--------------+----------------+---------------+------------------+----------------------+--+
| 10002        | rahul          | 23            | 250000           | BIGDATA              |
| 2            | sakshi         | 22            | 60000            | HR                   |
| 10001        | rajesh         | 29            | 50000            | BIGDATA              |
| 10003        | dinesh         | 35            | 70000            | BIGDATA              |
| 1            | aarti          | 28            | 55000            | HR                   |
| 3            | mahesh         | 25            | 25000            | HR                   |
+--------------+----------------+---------------+------------------+----------------------+--+
```

## distribute by & sort by
```sql
#set the number of reducers to 2
0: jdbc:hive2://localhost:10000> set mapred.reduce.tasks=2;

0: jdbc:hive2://localhost:10000> select * from employee distribute by id sort by salary;
+--------------+----------------+---------------+------------------+----------------------+--+
| employee.id  | employee.name  | employee.age  | employee.salary  | employee.department  |
+--------------+----------------+---------------+------------------+----------------------+--+
| 2            | sakshi         | 22            | 60000            | HR                   |
| 10002        | rahul          | 23            | 250000           | BIGDATA              |
| 3            | mahesh         | 25            | 25000            | HR                   |
| 10001        | rajesh         | 29            | 50000            | BIGDATA              |
| 1            | aarti          | 28            | 55000            | HR                   |
| 10003        | dinesh         | 35            | 70000            | BIGDATA              |
+--------------+----------------+---------------+------------------+----------------------+--+

```
## cluster by
只针对某个列的 distribute by col + sort by col;
```sql
#set the number of reducers to 2
0: jdbc:hive2://localhost:10000> set mapred.reduce.tasks=2;

0: jdbc:hive2://localhost:10000> select * from employee distribute by id sort by id;
+--------------+----------------+---------------+------------------+----------------------+--+
| employee.id  | employee.name  | employee.age  | employee.salary  | employee.department  |
+--------------+----------------+---------------+------------------+----------------------+--+
| 2            | sakshi         | 22            | 60000            | HR                   |
| 10002        | rahul          | 23            | 250000           | BIGDATA              |
| 1            | aarti          | 28            | 55000            | HR                   |
| 3            | mahesh         | 25            | 25000            | HR                   |
| 10001        | rajesh         | 29            | 50000            | BIGDATA              |
| 10003        | dinesh         | 35            | 70000            | BIGDATA              |
+--------------+----------------+---------------+------------------+----------------------+--+
```

```sql
#set the number of reducers to 2
0: jdbc:hive2://localhost:10000> set mapred.reduce.tasks=2;

0: jdbc:hive2://localhost:10000> select * from employee cluster by id;
+--------------+----------------+---------------+------------------+----------------------+--+
| employee.id  | employee.name  | employee.age  | employee.salary  | employee.department  |
+--------------+----------------+---------------+------------------+----------------------+--+
| 2            | sakshi         | 22            | 60000            | HR                   |
| 10002        | rahul          | 23            | 250000           | BIGDATA              |
| 1            | aarti          | 28            | 55000            | HR                   |
| 3            | mahesh         | 25            | 25000            | HR                   |
| 10001        | rajesh         | 29            | 50000            | BIGDATA              |
| 10003        | dinesh         | 35            | 70000            | BIGDATA              |
+--------------+----------------+---------------+------------------+----------------------+--+
```

## 上述四种排序对应的 map reduce 实现方式
### sort by
mapreduce 实现局部排序
```python
--------------------Map Phase---------------------------------|------------------------------Reduce Phase----------------------
(map    ->    partition    ->    sort    ->  |-----------|)   ->    (shuffle    ->    merge sort    ->    group    ->    reduce)
 |              |                |           |split merge|                              |                  |               |
 |              |                |           |sort       |                              |                  |               |
map()       Partitioner     SortComparator                                         SortComparator    GroupComparator    reduce()

```
其实 MapReduce 本身就已经是默认局部排序了， 只要实现自己的 key 的 Partitioner SortComparator 和 GroupComparator 即可自动 reduce 内部排序

### order by
mapreduce 实现全序排序

第一种方法是在正常的排序完成之后, 对第一阶段每一个 reducer 结果再次构造 MapReduce，但是该阶段只设置 一个 reducer， 那最后合并到一个reducer文件即可，但是当数量量很大，这个会导致某个reducer 压力很大；

第二种方法就是，在操作partition 的时候， 让 reducer 组间 有序， 即 根据需要改写 Partitioner， 根据 key 范围进行划分不同的 reducer，保证 reducer 之间有序； 这个可以自己实现，但是hadoop 已经提供了一个 TotalOrderPartitioner 了， 采用 trie 树 采样达到负载均衡；

### distribute by
MapReduce 实现组间排序
只需要实现自定义的 Partitioner 即可实现组间排序；

### cluster by
MapReduce 实现 聚集排序
同时实现自定义 Partitioner SortComparator GroupComparator 即可；

### secondary sort
MapReduce 多级排序实现；

自定义复合Key值 CompositeKey， 按需要实现 自定义 Partitioner SortComparator GroupComparator