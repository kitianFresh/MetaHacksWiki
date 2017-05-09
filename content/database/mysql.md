---
title: "mysql"
date: 2017-05-09 21:40
---

# MySQL 表定义范例
用于爬取 [icd10data](http://www.icd10data.com/ICD10CM/Codes) 中的数据，mysql 数据库模式如下：
```
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
```
mysqldump -u[username] -p [databasename] > databasename.sql
```

备份数据库中的某些表
```
mysqldump -u[username] -p [databasename] [tablename1] [tablename2] > dumps.sql
```

恢复数据库
```
mysql -u[username] -p [databasename] < databasename.sql
``` 

另一种备份方式是数据库元数据和数据表分开在不同的文件中
```
mysqldump -u[username] -p -t -T/path/to/directory [database] --fields-enclosed-by=\" --fields-terminated-by=,
```

只获取数据库元数据备份或者表元数据备份
```
mysqldump --no-data -u someuser -p mydatabase
mysqldump -d -u someuser -p mydatabase table1 table2 table3
```

# 导出 Mysql 数据 到 csv
导出到csv时，需要注意分隔符和转义字符的问题；
 - Excel saves \r\n for line separators.
 - Excel saves \n for newline characters within column data
 - Have to replace \r\n inside your data first otherwise Excel will think its a start of the next line.

```
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
```
The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
```

解决办法是Mysql默认有保护措施，只能输出文件到特定目录，通过一下命令查看目录
```
mysql> SELECT @@global.secure_file_priv;
```
