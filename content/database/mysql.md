---
title: "mysql"
date: 2017-05-09 21:40
---

# MySQL 表定义范例
用于爬取 [icd10data](http://www.icd10data.com/ICD10CM/Codes) 中的数据，mysql 数据库模式如下：
```sql
drop database if exists `icdInfo`;
create database `icdInfo`;
use `icdInfo`;

drop table if exists `Tbl_ICD10CM`;
create table `Tbl_ICD10CM` (
    `ClinicalId` int unsigned auto_increment not null,
    `DiagnosisCode` varchar(255) not null default '00' comment '疾病诊断码',
    `DiseaseName` varchar(255) comment '疾病名称',
    `ClinicalInfo` text comment '临床信息描述',
    `CodeIntervalUrl` varchar(255) not null comment '抓取该信息的url',
    `ScrapedTime` timestamp not null comment '更新时间',
    primary key (`ClinicalId`)
) ENGINE=InnoDB
```


# mysql 数据库备份
备份整个数据库

```sql
mysqldump -u[username] -p [databasename] > databasename.sql
```

备份数据库中的某些表
```sql
mysqldump -u[username] -p [databasename] [tablename1] [tablename2] > dumps.sql
```

恢复数据库
```sql
mysql -u[username] -p [databasename] < databasename.sql
``` 

另一种备份方式是数据库元数据和数据表分开在不同的文件中
```sql
mysqldump -u[username] -p -t -T/path/to/directory [database] --fields-enclosed-by=\" --fields-terminated-by=,
```

只获取数据库元数据备份或者表元数据备份
```sql
mysqldump --no-data -u someuser -p mydatabase
mysqldump -d -u someuser -p mydatabase table1 table2 table3
```

# 导出 Mysql 数据 到 csv
导出到csv时，需要注意分隔符和转义字符的问题；
 - Excel saves \r\n for line separators.
 - Excel saves \n for newline characters within column data
 - Have to replace \r\n inside your data first otherwise Excel will think its a start of the next line.

```sql
USE `icdInfo`;
SELECT
DiagnosisCode,
REPLACE( IFNULL(ClinicalInfo, ''), '\r\n' , '\n' ) AS ClinicalInfo
FROM Tbl_ICD10CM
INTO OUTFILE '/var/lib/mysql-files/icd.csv' 
FIELDS TERMINATED BY ',' ENCLOSED BY '"' ESCAPED BY '"'
LINES TERMINATED BY '\r\n';

```

出错
```sql
The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
```

解决办法是Mysql默认有保护措施，只能输出文件到特定目录，通过一下命令查看目录
```sql
mysql> SELECT @@global.secure_file_priv;
```

# MySQL大数据统计
## MySQL 聚合函数
[MySQL 聚合函数](https://dev.mysql.com/doc/refman/5.7/en/group-by-functions.html)可以理解成 `Map/Reduce` 中的　`reduce` 函数，就是将一个集合经过运算之后降维成一个数．常见的聚合函数就是我们数学统计上的个数(不同值的个数)，求和，均值，最大，最小，方差等．这些函数SQL 直接提供的．

 - COUNT()/COUNT(DISTINCT)
 - SUM()/SUM(DISTINCT)
 - AVG()/AVG(DISTINCT)
 - MAX()
 - MIN()
 - STD()
 - GROUP_CONCAT()

```sql
mysql> SELECT Class, COUNT(Age) FROM students GROUP BY Class;

mysql> SELECT Class, COUNT(DISTINCT Age) FROM students GROUP BY Class;

mysql> SELECT Class, MAX(Age) FROM students GROUP BY Class;

mysql> SELECT Class, MIN(Age) FROM students GROUP BY Class;

mysql> SELECT Class, SUM(Age) FROM students GROUP BY Class;

mysql> SELECT Class, AVG(Age) FROM students GROUP BY Class;

mysql> SELECT Class, SUM(DISTINCT Age) FROM students GROUP BY Class;

mysql> SELECT Class, AVG(DISTINCT Age) FROM students GROUP BY Class;

```
**中位数Mode**, 需要我们做更复杂的操作，

聚合函数运用在一个集合数据上，因此对于查询出来的表或者分组都可用的，运用特别多的就是和　GROUP BY 一起使用．因为 GROUP BY 是无法打印出来每一个组的所有记录的！此时还有一个聚合函数　`GROUP_CONCAT()`;此函数可以将每一组的某个字段值连接成一个字符串字段；
```sql
GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR str_val])

```
例如，查询每一个班上所有同学的Id, 并按照年龄大小降序．
```sql
mysql> SELECT Class, GROUP_CONCAT(Id ORDER BY Age DESC) FROM students GROUP BY Class;
+-------+------------------------------------------------------------------+
| Class | GROUP_CONCAT(Id ORDER BY Age DESC)                               |
+-------+------------------------------------------------------------------+
|     1 | 95,87,86,73,89,10,66,47,55,1,52,2,21,15,20,9,25,85,98            |
|     2 | 34,64,51,43,38,18,69,94,29,67,68,45,13,32,58,31,65,33,92,49,8,83 |
|     3 | 37,30,48,81,74,72,100,56,53,36,44,23,70,12,59,24,35,60,11,84     |
|     4 | 62,46,54,50,57,5,93,63,41,40,78,14,4,82,99,26,97,39,28,80        |
|     5 | 7,96,71,42,3,17,79,19,91,75,22,90,6,61,88,76,16,27,77            |
+-------+------------------------------------------------------------------+
5 rows in set (0.00 sec)
```
## 统计分组排序后的 TOP N
### 使用相关子查询
```sql
SELECT Id, Name, Class, Age 
FROM students s 
WHERE
  (
   SELECT COUNT(*)
   FROM students
   WHERE
   students.Class = s.Class AND students.Age >= s.Age
  ) <= 5
GROUP BY Class, Age DESC, Id;

+----+---------------+-------+------+
| Id | Name          | Class | Age  |
+----+---------------+-------+------+
| 86 | Bernd Baker   |     1 |   45 |
| 87 | Roger Smith   |     1 |   45 |
| 95 | Eve Moore     |     1 |   45 |
| 73 | John Baker    |     1 |   43 |
| 34 | Eve Smith     |     2 |   45 |
| 64 | Eve Smith     |     2 |   45 |
| 43 | Eve Baker     |     2 |   44 |
| 51 | Bernard Smith |     2 |   44 |
| 38 | John Smith    |     2 |   43 |
| 37 | Yvonne Miller |     3 |   45 |
| 30 | Eve Baker     |     3 |   44 |
| 48 | Eve Baker     |     3 |   43 |
| 81 | Bill Miller   |     3 |   40 |
| 62 | John Smith    |     4 |   43 |
| 46 | Bernd Smith   |     4 |   41 |
| 54 | Yvonne Baker  |     4 |   41 |
| 50 | John Baker    |     4 |   40 |
| 57 | John Smith    |     4 |   38 |
|  7 | Simone Miller |     5 |   44 |
| 96 | Eve Looper    |     5 |   42 |
|  3 | Bill Smith    |     5 |   41 |
| 42 | Zoe Smith     |     5 |   41 |
| 71 | Eve Smith     |     5 |   41 |
+----+---------------+-------+------+
23 rows in set (0.01 sec)

```
以上查询的意思就是，对于 students 表中的每一个学生，看他是不是改组的 TOP 5, 其实就是看有多少个和 s.Class 相同的学生他的年龄比　s.Age 大，如果个数小于 N 个，　就表明这个学生是　TOP N 中的，　因为　TOP N 中的记录，比他还大的必定少于 N 个；　但是这个版本的缺陷可以从结果看出来，　他无法正确统计值相同的情况！

### UNION 集合并
```sql
mysql> (select * from students where Class = 1 order by Age desc limit 5)
    -> union all
    -> (select * from students where Class = 2 order by Age desc limit 5)
    -> union all
    -> (select * from students where Class = 3 order by Age desc limit 5)
    -> union all
    -> (select * from students where Class = 4 order by Age desc limit 5)
    -> union all
    -> (select * from students where Class = 5 order by Age desc limit 5)
    -> ;

```
方法就是单独查询每一类别的TOP N, 然后使用　UNION　并集；

### User Defined Varibles
```sql

SELECT Id, Name, Class, Age
FROM
(
    SELECT Id, Name, Class, Age,
        @ClassRank := IF(@CurrentClass = Class, @ClassRank + 1, 1) As ClassRank,
        @CurrentClass := Class
    FROM students
    ORDER BY Class, Age DESC
) ranked
WHERE ClassRank <= 5
ORDER BY Class, Age DESC;
```
该SQL 的意思就是，给每一个类别中的学生设置rank, 他是通过已经排好序的　Age, 然后计算出每一个学生的 RANK, 最后取出排序在　前　Ｎ 个的学生即可； 变量字段　@ClassRank 和　@ CurrentClass 开始都是空，然后通过不断的赋值更新；由于学生是排好序的，因此后一个和前一个是同类，则每一个rank 就+1, 不是同一类时候就置１，然后开始下一个类的 rank 操作；

## MySQL Limit 分页查询
`limit offset, row_count`, `limit row_count` 等价于　`limit 0, row_count`; 分页是常见的场景，分页实现也有不同的版本; 简单的就是使用　`limit`. 分页一般是按照某种排序标准进行分页，例如学生的数学成绩，或者入学时间等；

以下语句按照时间顺序查询 第`21 ~ 30`条记录 即第三页，每页10条记录；
```sql
mysql> SELECT * FROM students ORDER BY Registry DESC LIMIT 20, 10;
+----+--------------+-------+------+------+------+----------+---------+---------------------+
| Id | Name         | Class | Age  | Sex  | Math | Computer | Physics | Registry            |
+----+--------------+-------+------+------+------+----------+---------+---------------------+
| 79 | Frank Miller |     5 |   39 | 女   |   43 |       24 |      78 | 2017-05-17 22:24:02 |
| 80 | Paul Smith   |     4 |   22 | 女   |   56 |       92 |      37 | 2017-05-17 22:24:02 |
| 77 | Simone Baker |     5 |   24 | 男   |   43 |       43 |       4 | 2017-05-17 22:24:00 |
| 78 | John Baker   |     4 |   32 | 男   |   86 |       56 |      50 | 2017-05-17 22:24:00 |
| 75 | Laura Smith  |     5 |   31 | 女   |   66 |        7 |      29 | 2017-05-17 22:23:58 |
| 76 | Eve Smith    |     5 |   26 | 女   |   59 |        5 |      81 | 2017-05-17 22:23:58 |
| 73 | John Baker   |     1 |   43 | 男   |   54 |       94 |       7 | 2017-05-17 22:23:56 |
| 74 | John Smith   |     3 |   36 | 男   |   58 |       13 |      64 | 2017-05-17 22:23:56 |
| 71 | Eve Smith    |     5 |   41 | 女   |   31 |        8 |      21 | 2017-05-17 22:23:54 |
| 72 | Paul Singer  |     3 |   36 | 男   |    9 |       11 |      52 | 2017-05-17 22:23:54 |
+----+--------------+-------+------+------+------+----------+---------+---------------------+
10 rows in set (0.00 sec)

```

## MySQL 

```sql
select isnull(item1,0)+isnull(item2,0)+isnull(item3,0) as itemAdd, usr from usr_table
select sum(isnull(item1,0)+isnull(item2,0)+isnull(item3,0))as itemSum  from usr_table
```



