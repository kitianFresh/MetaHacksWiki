---
title: "mongodb"
date: 2017-04-23 21:26
---

### Mongodb 安装
安装[mongoDB](https://docs.mongodb.com/manual/installation/)
```
// ubuntu 16.04
echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
// ubuntu 14.04
echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

//国外repo直接安装太慢，因此这里把你的Ubuntu软件源更换为aliyun或者中科大的。
//将上面的 http://repo.mongodb.org 更换为 http://mirrors.aliyun.com/mongodb
echo "deb http://mirrors.aliyun.com/mongodb/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

sudo apt-get update
sudo apt-get install -y mongodb-org
sudo service mongodb started
sudo mongo

```

#### 参考
 - [官网](https://docs.mongodb.com/manual/installation/)
 - [install-mongodb-on-ubuntu-16.04](https://www.howtoforge.com/tutorial/install-mongodb-on-ubuntu-16.04/)

#### 国外镜像安装过慢的方法
 - [Ubuntu16.04使用阿里云镜像安装Mongodb](http://www.linuxdiyf.com/linux/26151.html)

### Mongodb的python客户端开发
安装[python driver](https://docs.mongodb.com/getting-started/python/client/)
`pip install pymongo`

### Pymongo fast tutorial

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from pymongo import MongoClient

# CRUD
# create
# 1. create a connection
client = MongoClient("mongodb://localhost:27017")
#default connect to mongodb://localhost:27017
#client = MongoClient()

# 2. access database objects, remote database object assign to local db
db = client.test
#db = client['test'] dictionary-style

# 3. access collection objects
coll = db.restaurants
#coll = db['restaurants']

# update
from datetime import datetime
'''Python
The operation returns an InsertOneResult object, 
which includes an attribute inserted_id that contains the _id of the inserted document. 
Access the inserted_id attribute:

result = coll.insert_one(
    {
        "address": {
            "street": "2 Avenue",
            "zipcode": "10075",
            "building": "1480",
            "coord": [-73.9557413, 40.7720266]
        },
        "borough": "Manhattan",
        "cuisine": "Italian",
        "grades": [
            {
                "date": datetime.strptime("2014-10-01", "%Y-%m-%d"),
                "grade": "A",
                "score": 11
            },
            {
                "date": datetime.strptime("2014-01-16", "%Y-%m-%d"),
                "grade": "B",
                "score": 17
            }
        ],
        "name": "Vella",
        "restaurant_id": "41704620"

    }
)
print(result.inserted_id)
'''

# read
# query by a top level field
cursor = db.restaurants.find({"borough": "Manhattan"})
for document in cursor:
    print(document)

# query by a field in an embedded document,use dot notation
cursor = db.restaurants.find({"address.zipcode": "10075"})
for document in cursor:
    print(document)
```


## MongoDB 数据处理

### 基本操作
```javascript
db.info.find({"icd_code": {$ne: null}}).count()
```
### 导入导出Json
```javascript
mongoexport --db <database-name> --collection <collection-name> --out output.json

mongoimport --db <database-name> --collection <collection-name> --file input.json
```

### 聚合删除
```javascript
// 分组计算个数并倒序
db.info.aggregate([{$group:{_id:"$chinese_name", count: { $sum: 1 }}},{$sort: {count: -1}}])

// 保证索引 unique 
db.info.ensureIndex({"chinese_name": 1}, {unique:true, dropDups:true})
{
	"ok" : 0,
	"errmsg" : "E11000 duplicate key error collection: JBBK.info index: chinese_name_1 dup key: { : \"B疱疹病毒感染\" }",
	"code" : 11000
}
```

// 打印所有索引

```javascript
db.getCollectionNames().forEach(function(collection) {    indexes = db[collection].getIndexes();    print("Indexes for " + collection + ":");    printjson(indexes); });
```

// 某个 Collection的索引
```javascript
db.collection.getIndexes()
```
MongoDB 直接支持 JavaScript脚本, 因此 使用 ` mongo JBBK < remove_dups.js` 可以直接执行, 他可以用来删除 Collection 中重复的行;

```javascript
\\ remove_dups.js
var duplicates = [];

db.getCollection('info').aggregate([  
  { $match: { 
      chinese_name: { $ne: ''}
  }},
  { $group: { 
      _id: { chinese_name: "$chinese_name"},
      count: { $sum: 1},
      dups: { $push: "$_id"}, 

  }}, 
  { $match: { 
      count: { $gt: 1}
  }}
])               
.forEach(function(doc) {
    doc.dups.shift();      
    doc.dups.forEach( function(dupId){ 
        duplicates.push(dupId);
        }
    )    
})


db.getCollection('info').remove({_id:{$in:duplicates}})
```


```javascript
db.userInfo.aggregate([

    {

        $group: { _id: {userName: '$userName',age: '$age'},count: {$sum: 1},dups: {$addToSet: '$_id'}}

    },

    {

        $match: {count: {$gt: 1}}

    }

]).forEach(function(doc){

    doc.dups.shift();

    db.userInfo.remove({_id: {$in: doc.dups}});

})

1.根据userName和age分组并统计数量，$group只会返回参与分组的字段，使用$addToSet在返回结果数组中增加_id字段

2.使用$match匹配数量大于1的数据

3.doc.dups.shift();表示从数组第一个值开始删除；作用是踢除重复数据其中一个_id，让后面的删除语句不会删除所有数据

4.使用forEach循环根据_id删除数据

$addToSet 操作符只有在值没有存在于数组中时才会向数组中添加一个值。如果值已经存在于数组中，$addToSet返回，不会修改数组。
```

 - [从mysql迁移数据至mongoDB](https://my.oschina.net/amoszhou/blog/675766)