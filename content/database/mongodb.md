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
```
pip install pymongo
```
fast tutorial
```
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