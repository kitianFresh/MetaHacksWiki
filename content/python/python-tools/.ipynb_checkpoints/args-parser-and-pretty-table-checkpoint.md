---
title: "args-parser-and-pretty-table"
date: 2018-02-24 10:20
---
# args-parser
```python
# -*- coding: utf8 -*-
import MySQLdb
import argparse

#### 参数调用说明
parser = argparse.ArgumentParser()
parser.add_argument('-a', '--host', help='mysql host', default='localhost')
parser.add_argument('-t', '--port', help='mysql port', type=int, default=3306)
parser.add_argument('-u', '--user',
                    help='mysql user', default="root")
parser.add_argument('-p', '--passwd', help='mysql password', required=True)
parser.add_argument('-d', '--db',
                    help="mysql database", type=str, required=True)
parser.add_argument('-c', '--charset', help='mysql connect charset', default='utf8')
parser.add_argument('-z', '--zone', help='image zone where you want to deleted', action='append', required=True)
args = parser.parse_args()
def getMysqlConn(passwd, db, host="localhost", port=3306, user="root", charset='utf8'):
    conn = MySQLdb.connect(host=host, port=port, user=user, passwd=passwd, db=db, charset=charset)
    return conn


def delete_image_properties(images_to_delete, passwd, db, host="localhost", port=3306, user="root", charset='utf8'):
    conn = getMysqlConn(passwd, db, host, port, user, charset)
    try:
        cur = conn.cursor()
        format_strings = ','.join(['%s'] * len(images_to_delete))
        query = "DELETE FROM `image_properties` WHERE `image_id` IN (%s)" % format_strings
        cur.execute(query, tuple(map(str, images_to_delete)))
        conn.commit()
    except Exception as e:
        import traceback
        print traceback.format_exc()
    finally:
        conn.close()


def delete_image_members(images_to_delete, passwd, db, host="localhost", port=3306, user="root", charset='utf8'):
    conn = getMysqlConn(passwd, db, host, port, user, charset)
    try:
        cur = conn.cursor()
        placeholder = '%s'
        placeholders = ', '.join(placeholder for unused in images_to_delete)
        query = 'DELETE FROM `image_members` WHERE `image_id` IN (%s)' % placeholders
        cur.execute(query, tuple(map(str, images_to_delete)))
        conn.commit()
    except Exception as e:
        import traceback
        print traceback.format_exc()
    finally:
        conn.close()

def delete_image_tags(images_to_delete, passwd, db, host="localhost", port=3306, user="root", charset='utf8'):
    conn = getMysqlConn(passwd, db, host, port, user, charset)
    try:
        cur = conn.cursor()
        placeholder = '%s'
        placeholders = ', '.join(placeholder for unused in images_to_delete)
        query = 'DELETE FROM `image_tags` WHERE `image_id` IN (%s)' % placeholders
        cur.execute(query, tuple(map(str, images_to_delete)))
        conn.commit()
    except Exception as e:
        import traceback
        print traceback.format_exc()
    finally:
        conn.close()

def delete_images(images_to_delete, passwd, db, host="localhost", port=3306, user="root", charset='utf8'):
    conn = getMysqlConn(passwd, db, host, port, user, charset)
    try:
        cur = conn.cursor()
        placeholder = '%s'
        placeholders = ', '.join(placeholder for unused in images_to_delete)
        query = 'DELETE FROM `images` WHERE `id` IN (%s)' % placeholders
        cur.execute(query, tuple(map(str, images_to_delete)))
        conn.commit()
    except Exception as e:
        import traceback
        print traceback.format_exc()
    finally:
        conn.close()


if __name__ == '__main__':

    host = args.host
    port = args.port
    user = args.user
    passwd = args.passwd
    db = args.db
    charset = args.charset
    zones = args.zone
    print host
    print port
    print user
    print passwd
    print db
    print charset
    print zones
    conn = getMysqlConn(passwd, db, host, port, user, charset)
    try:
        cur = conn.cursor()
        queryStr = "select `image_id` from `image_properties` where `deleted` = 0 and "
        subQuery = []
        for zone in zones:
            subQuery.append("(`name` = 'zone' and `value` = %s)")
        subStr = "(" + " or ".join(subQuery) + ")"
        queryStr = queryStr + subStr + " order by `updated_at` desc"
        print queryStr
        cur.execute(queryStr, zones)
        images_to_delete = cur.fetchall()
        imgs = []
        for img in images_to_delete:
            imgs.append(img[0])
        print imgs, len(imgs)
        images_to_delete = imgs
        if len(images_to_delete) > 0:
            delete_image_properties(images_to_delete, passwd, db, host, port, user, charset)
            delete_image_members(images_to_delete, passwd, db, host, port, user, charset)
            delete_image_tags(images_to_delete, passwd, db, host, port, user, charset)
            delete_images(images_to_delete, passwd, db, host, port, user, charset)
    except Exception as e:
        import traceback
        print traceback.format_exc()
    finally:
        conn.close()

```

## args-tab-hint
```python
def arg(*args, **kwargs):
    def _decorator(func):
        # Because of the sematics of decorator composition if we just append
        # to the options list positional options will appear to be backwards.
        func.__dict__.setdefault('arguments', []).insert(0, (args, kwargs))
        return func
    return _decorator

@arg('id', 'name', metavar='<SERVER_ID>', help='ID of virtual server to get metadata info')
@arg('--field', metavar='<METADATA_FIELD>', help='Field name of metadata')
def do(*args, **kwargs):
    print 'ok'
do.__dict__
# do.__dict__.setdefault('argument',[]).insert(0, (1,2))
# do.__dict__.setdefault('argument',[]).insert(0, (2,3))

```

# Terminal display table
pip install prettytable
```python
from prettytable import PrettyTable
row = PrettyTable()
row.field_names = ['Property', 'Value']
maps = {
    'name': 'kiki',
    'type': 'hacker',
    'id': '0x70'
}
for (k, v) in maps.items():
    row.add_row([k,v])
print row
print row.get_string(sortby='Property')
```
