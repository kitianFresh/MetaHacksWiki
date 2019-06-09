---
title: "mysql"
date: 2017-05-09 21:40
---
# 在centos上安装MySQL
```
sudo yum install Percona-Server-server-55.x86_64
sudo service mysql start
sudo chkconfig --level 2345 mysql on
mysql_secure_installation
```

# 重置MySQL root 密码
1. 停止MySQL服务 `sudo service mysql stop`

2. 设置MySQL安全模式 `sudo mysqld_safe --skip-grant-tables --skip-networking &`

3. 无密码进入MySQL `mysql -u root`

4. 重置密码
```sql
mysql> use mysql;  
mysql> update user set password=PASSWORD("mynewpassword") where User='root';  
mysql> flush privileges; 
```
如果 update 不成功，比如报错说没有 password 字段，看看自己的MYSQL版本，新版本 5.7 密码字段叫 `authentication_string`
```sql
UPDATE user SET authentication_string=PASSWORD("NEWPASSWORD") WHERE User='root';
```
5. 重启MySQL服务 `sudo service mysql restart`

6. 苹果上重启后可能还遇到问题`ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.`
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'YourNewPass';
```

# MYSQL 创建用户并授权
创建
```sql
insert into mysql.user(Host,User,Password) values("localhost","cron",password("cron"));
flush privileges;
```
以上方法过时了。。。采用
`CREATE USER 'cron'@'localhost' IDENTIFIED BY 'cron';`
授权
```sql
create database cron;
grant all privileges on cron.* to cron@localhost identified by 'cron';
# grant select,update on cron.xxx, cron.yyy to cron@localhost identified by 'cron(password)';
flush privileges;
```
以上方法不对，去掉后面的 identified by 

json invalid character ':' after top-level value
json 字符串紧密相连，不能出现其他多余空格

ERROR 3140 (22032) at line 20: Invalid JSON text: The document root must not be followed by other values. at position 8 in value for column clusters.master.
```sql
'{"master":{"apiserver":{"leader":"","logdir":"","nodes":[]},"controllermanager":{"leader":"gh-inf-hulkqa-kubecontroller-test01.corp.sankuai.com","logdir":"","nodes":[]},"etcd":{"leader":"","logdir":"","nodes":[]},"scheduler":{"leader":"gh-inf-hulkqa-kubescheduler-test01.corp.sankuai.com","logdir":"","nodes":[]}}}'
```
第二，json字符串在mysql插入的时候，比如是'{}'在最外层


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

# mysql 查看索引
```sql
show index from table_name;

ALTER TABLE disks_tbl ADD COLUMN `auto_backup` VARCHAR(32) CHARACTER SET 'ascii';
ALTER TABLE mebs_snapshots_tbl DROP COLUMN `auto_take`;
ALTER TABLE mebs_snapshots_tbl ADD COLUMN `snapshot_type` VARCHAR(32) CHARACTER SET 'ascii' NOT NULL DEFAULT '';

ALTER TABLE disks_tbl ADD INDEX ix_disks_tbl_auto_backup (`auto_backup`);
ALTER TABLE mebs_snapshots_tbl ADD INDEX ix_mebs_snapshots_tbl_snapshot_type (`snapshot_type`);

CREATE TABLE `mebs_snapshot_strategy_disks_tbl` (
`created_at` datetime NOT NULL,
`updated_at` datetime NOT NULL,
`deleted_at` datetime DEFAULT NULL,
`deleted` tinyint(1) NOT NULL,
`row_id` bigint(20) NOT NULL AUTO_INCREMENT,
`mebs_snapshot_strategy_id` varchar(36) CHARACTER SET ascii NOT NULL,
`disk_id` varchar(36) CHARACTER SET ascii NOT NULL,
`cron_job_id` varchar(36) CHARACTER SET ascii DEFAULT NULL,
`open` tinyint(1) NOT NULL,
PRIMARY KEY (`row_id`),
KEY `ix_mebs_snapshot_strategy_disks_tbl_deleted` (`deleted`),
KEY `ix_mebs_snapshot_strategy_disks_tbl_open` (`open`),
KEY `ix_mebs_snapshot_strategy_disks_tbl_mebs_snapshot_strategy_id` (`mebs_snapshot_strategy_id`),
KEY `ix_mebs_snapshot_strategy_disks_tbl_created_at` (`created_at`),
KEY `ix_mebs_snapshot_strategy_disks_tbl_disk_id` (`disk_id`),
KEY `ix_mebs_snapshot_strategy_disks_tbl_cron_job_id` (`cron_job_id`)
) ENGINE=InnoDB AUTO_INCREMENT=92 DEFAULT CHARSET=utf8mb4
```

# mysql sql语句结果导出
```sql
mysql -uroot -p1234 -Ne "use cron; select id from cron_jobs_tbl;" > /tmp/jobs.txt
insert into cron_logs_tbl(id,cron_job_id,created_at,updated_at) values(uuid(),"001d4865-127c-45ba-90de-6c1f74e5a782",now(),now());
```

# MySQL大数据统计Case Study
```sql
drop database if exists `StudentManager`;
create database `StudentManager`;
use `StudentManager`;

drop table if exists `students`;
create table `students` (
    `Id` int unsigned auto_increment not null,
    `Name` varchar(255) not null,
    `Class` int unsigned,
    `Age` int unsigned,
    `Sex` varchar(255),
    `Math` int,
    `Computer` int,
    `Physics` int,
    `Registry` timestamp not null,
    primary key (`Id`)
)
```

# SQL JOIN
## inner join
`SELECT <field_list> FROM TABLE_A A INNER JOIN TABLE_B B ON A.KEY = B.KEY`
求左右两边相同键值的笛卡尔积。举个例子，假设 A.a1 在左边有两条记录，`rec_num(A.a1) = 2`, B.a1 在右边有一条记录 `rec_num(B.a1) = 1`（一般是B的主键）, 那么连接之后，a1 记录就有 2 x 1 条记录。如果 A.a1 有 m 条记录， B.a1在右边有多条设为 n 条记录，则最后 a1 记录就有 m x n 条， 这就是笛卡尔积。即 左边的每一条和右边的每一条都得组合。这里做的笛卡尔积只对 m >= 1 && n >= 1 的时候才会加入最终的连接结果，这就是 inner join， 只包含 连接键值满足 m >= 1 && n >= 1 的笛卡尔积。

## left join
`SELECT <field_list> FROM TABLE_A A LEFT JOIN TABLE_B B ON A.KEY = B.KEY`
以左边基准，求左右两边相同键值的笛卡尔积。 这里的笛卡尔积必须包含左边表所有的键值，如果 `rec_num(A.a1) > 0 && rec_num(B.a1) = 0`，这在 inner join 中就会被去除，但是在 left join 中，会保留这些 A.a1 记录，连接起来的右边表的值都设置为默认 null。`A.a1, A.a2,..., B.null, B.null...`

## right join
`SELECT <field_list> FROM TABLE_A A RIGHT JOIN TABLE_B B ON A.KEY = B.KEY`
和left join 相反，以右边基准，求左右两边相同键值的笛卡尔积。 这里的笛卡尔积必须包含右边表所有的键值，如果 `rec_num(A.a1) = 0 && rec_num(B.a1) > 0`，这在 inner join 中会被去除，但是在 left join 中，会保留这些 B.a1 记录，连接起来的左边表的值都设置为默认 null。 `A.null, A.null, ... A.null, B.a1, B.a2, B.a3...`

## outer join
`SELECT <field_list> FROM TABLE_A A FULL OUTER JOIN TABLE_B B ON A.KEY = B.KEY`
这个就是 left join + right join。 左右两边的键值都保留。

## left join excluding inner join
`SELECT <field_list> FROM TABLE_A A LEFT JOIN TABLE_B B ON A.KEY = B.KEY WHERE B.KEY IS NULL`

## right join excluding inner join
`SELECT <field_list> FROM TABLE_A A RIGHT JOIN TABLE_B B ON A.KEY = B.KEY WHERE A.KEY IS NULL`

##outer join excluding inner join 
`SELECT <field_list> FROM TABLE_A A FULL OUTER JOIN TABLE_B B ON A.KEY = B.KEY WHERE A.KEY IS NULL OR B.KEY IS NULL`

以上在SQLAlchemy中的对应关系
- [Visual Representation of SQL Joins](https://www.codeproject.com/Articles/33052/Visual-Representation-of-SQL-Joins)

# SQL 执行顺序
```sql
SELECT DISTINCT column, AGG_FUNC(column_or_expression), …
FROM mytable
    JOIN another_table
      ON mytable.column = another_table.column
    WHERE constraint_expression
    GROUP BY column
    HAVING constraint_expression
    ORDER BY column ASC/DESC
    LIMIT count OFFSET COUNT;
```
根据数据库不同，其实内部的执行顺序是不确定的，同一中数据库不同查询语句也有可能不同，因为有sql优化器和执行计划，但是有一个大致默认的顺序，
FROM & JOIN -> WHERE -> GROUP BY -> HAVING -> SELECT -> DISTINCT -> ORDER BY -> LIMIT/OFFSET
- [Order of execution of a Query](https://sqlbolt.com/lesson/select_queries_order_of_execution)

## MySQL 聚合函数
[MySQL 聚合函数](https://dev.mysql.com/doc/refman/5.7/en/group-by-functions.html)可以理解成 `Map/Reduce` 中的　`reduce` 函数，就是将一个集合经过运算之后降维成一个数．常见的聚合函数就是我们数学统计上的个数(不同值的个数)，求和，均值，最大，最小，极差，标准差，方差等．这些函数 SQL 直接提供的．[Statistical functions in MySQL](http://www.xarg.org/2012/07/statistical-functions-in-mysql/)

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

 - 均值
 - 中位数  --将数据按大小顺序排列后处于中间位置的数。当中间剩余为2个数字时，取其算数平均数。
 - 众数    --数据中出现次数最多的数（所占比例最大的数），一组数据中，可能存在多个众数，也可能不存在众数。
 - 极差　--最大值和最小值的差，衡量数据范围
 - 标准差　-- 距离平均数的距离，数据分布距离均值的带宽
 - 方差　-- 标准差的平方
 - 偏度
 - 峰度


优缺点

||含义|优点|	缺点|
|--:|--:|--:|--:|
|均值|AVG|充分利用所有数据，适用性强|容易受到极端值影响|
|中位数|MEDIUM|不受极端值影响	       |缺乏敏感性      |
|众数|MODE|当数据具有明显的集中趋势时，代表性好；不受极端值影响|缺乏唯一性：可能有一个，可能有两个，可能一个都没有|

**众数Mode**, 需要我们做更复杂的操作，寻找某一个组的众数．
```sql

SELECT * FROM (SELECT Class, Age, COUNT(Age) As Count FROM students WHERE Class = 1 GROUP BY Age) AS temp WHERE Count IN (SELECT MAX(Count) FROM (SELECT Class, Age, COUNT(Age) As Count FROM students WHERE Class = 1 GROUP BY Age) AS temp);
```
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
以上查询的意思就是，对于 students 表中的每一个学生，看他是不是该组的 TOP 5, 其实就是看有多少个和 s.Class 相同的学生他的年龄比　s.Age 大，如果个数小于 N 个，　就表明这个学生是　TOP N 中的，　因为　TOP N 中的记录，比他还大的必定少于 N 个；　但是这个版本的缺陷可以从结果看出来，　他无法正确统计值相同的情况！

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

## MYSQL time
unix 时间戳转换成时间
```sql
select id, from_unixtime(next_run_timestamp) from apscheduler_jobs_tbl;
```

```sql
select snapshot_id, name, snapshot_type, expired_days, created_at, utc_timestamp(), adddate(created_at, interval 
expired_days day) as expire, time_to_sec(timediff(adddate(created_at, interval expired_days day), utc_timestamp())) as lefted  from mebs_snapshots_tbl where expired_days!=-1 and status='mebs_snapshot_ready' and deleted=false order by lefted;
```

## MySQL 分页查询
### offset 分页查询
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
### value-based 分页查询
offset 版本的分页查询当面对大数据集合的时候，会非常低效，因为他总是先读出所有的数据，让后排序，这是非常耗时的。
### 分页查询优化
- [The art of pagination – Offset vs. value based paging](https://blog.novatec-gmbh.de/art-pagination-offset-vs-value-based-paging/)

- [Optimized Pagination using MySQL](https://www.xarg.org/2011/10/optimized-pagination-using-mysql/)
- [What’s wrong with pagination?](https://blog.jooq.org/2016/08/10/why-most-programmers-get-pagination-wrong/)

- [Pagination](http://mysql.rjweb.org/doc.php/pagination)

- [Finding the Best Match With a Top-N Query](http://blog.fatalmind.com/2010/09/29/finding-the-best-match-with-a-top-n-query/)

- [Pagination with OFFSET / FETCH : A better way](https://sqlperformance.com/2015/01/t-sql-queries/pagination-with-offset-fetch)

## MySQL列运算操作

```sql
select isnull(item1,0)+isnull(item2,0)+isnull(item3,0) as itemAdd, usr from usr_table
select sum(isnull(item1,0)+isnull(item2,0)+isnull(item3,0))as itemSum  from usr_table
```
## MySQL 索引优化
 1. 最左前缀匹配原则，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。
 2. =和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式
 3. 尽量选择区分度高的列作为索引,区分度的公式是count(distinct col)/count(*)，表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1，而一些状态、性别字段可能在大数据面前区分度就是0，那可能有人会问，这个比例有什么经验值吗？使用场景不同，这个值也很难确定，一般需要join的字段我们都要求是0.1以上，即平均1条扫描10条记录
 4. 索引列不能参与计算，保持列“干净”，比如from_unixtime(create_time) = ’2014-05-29’就不能使用到索引，原因很简单，b+树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。所以语句应该写成create_time = unix_timestamp(’2014-05-29’);
 5. 尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可

## MySQL 事务和锁
```sql
drop table if exists `productsStock`;
create table `productsStock` (
    `id` int(11) not null auto_increment,
    `name` varchar(255) default null,
    `desc` text(1024) default null,
    `stock` int(11) default null,
    `version` int(64) default 0;
    primary key (`id`),
    unique key `idx_name` (`name`) using hash
)
```
插入数据
```sql
INSERT INTO `productsStock` VALUES ('1', 'prod11', '', '1000');
INSERT INTO `productsStock` VALUES ('2', 'prod12', '', '1000');
INSERT INTO `productsStock` VALUES ('3', 'prod13', '', '1000');
INSERT INTO `productsStock` VALUES ('4', 'prod14', '', '1000');
INSERT INTO `productsStock` VALUES ('5', 'prod15', '', '1000');
INSERT INTO `productsStock` VALUES ('6', 'prod16', '', '1000');
INSERT INTO `productsStock` VALUES ('7', 'prod17', '', '1000');
INSERT INTO `productsStock` VALUES ('8', 'prod18', '', '1000');
INSERT INTO `productsStock` VALUES ('9', 'prod19', '', '1000');
```
### 乐观锁
MYSQL乐观锁的实现和CAS是一样的原理，需要对每条记录增加version列进行控制，通过loop的方式进行重试，在查询和更新的时候获得的是同一个版本则更新成功，否则说明已经被其他实例修改，重试，伪代码如下：
```java
int affectedCount = 0;
while (affectedCount == 0) {
    // 乐观锁，尝试修改，不成功则自旋重试
    Product product = execute("select * from productsStock where id=#{id}");
    if (product.getStock>0) {
        affectedCount = execute("update productsStock set stock=#{newStock}, version=#{product.version+1} where id=#{product.id} and version=#{product.version}");
    } else {
        return false;
    }
}
return true;
```
乐观锁适用于写的比较少，读多的场景。
### 悲观锁
悲观锁和synchronized一样，首先进行枷锁，然后进行更改操作，MySQL innodb 引擎在使用索引查询的时候会自动加行锁，即锁住某行记录。不需要加一个version列就可以做。
```java
// 悲观锁，上来就先锁住
Product product = execute("select * from productsStock where id=#{id} for update");
if (product.getStock>0) {
    execute("update productsStock set stock=#{newStock} where id=#{product.id}")
} else {
    return false;
}
```
悲观锁适用于写的多频繁，读的少的场景。因为写的多场景下，采用CAS乐观锁大部分都是失败，导致自旋浪费CPU。不如直接采用悲观锁。

1、如果对读的响应度要求非常高，比如证券交易系统，那么适合用乐观锁，因为悲观锁会阻塞读
2、如果读远多于写，那么也适合用乐观锁，因为用悲观锁会导致大量读被少量的写阻塞
3、如果写操作频繁并且冲突比例很高，那么适合用悲观写独占锁
 
### 行锁、表锁
不同的引擎和使用方式会导致MYSQL使用不同方式加锁，使用MYISAM引擎，都是表锁，而采用INNODB引擎，由于InnoDB 预设是Row-Level Lock，所以只有「明确」的指定主键，MySQL 才会执行Row lock (只锁住被选取的数据) ，否则MySQL 将会执行Table Lock (将整个数据表单给锁住)。

- [MySQL中的锁（表锁、行锁）](http://www.cnblogs.com/chenqionghe/p/4845693.html)
- [数据库：Mysql中“select ... for update”排他锁分析](https://blog.csdn.net/claram/article/details/54023216)
- [how-to-do-distributed-locking](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)
- [Questions about distributed locks](https://www.alibabacloud.com/forum/read-453)
- [ a simple distributed lock service](http://www.scs.stanford.edu/14au-cs244b/labs/projects/pu_gao_qu.pdf)
- [一种本地锁+分布式锁的实现](http://adamswanglin.com/wllock/)