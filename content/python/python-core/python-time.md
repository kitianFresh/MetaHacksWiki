---
title: "python-time"
date: 2017-12-06 23:08
---

[TOC]

# 时间和日期
编码乱码问题可能仅仅是中国程序员遇到的问题，但是时间时区问题可能是世界上所有的程序员都会遇到的问题！我们得搞清楚服务器的时间和用户所在地的时间关系，才能正确无误的进行编程！这个问题在进行定时任务设置的时候尤为明显！你需要考虑时区，和服务器所在的地的时区！
```python
import time

for name, val in vars(time).items():
    print '--------------------------------'
    print name, val

print __file__

import time
dir(time)
```
## 时间的流逝是绝对的，但是程序中的时间是相对的，不是绝对的！所以，时间差更有使用价值！
程序中有两个最重要的时间，使用最多且可以避免错误的是，utc时间和timestamp时间戳。第一个是时区时间的标准参考点，相当于物理坐标系中的原点，其他各区的时间以此为参考，根据所在时区即可计算；第二个就是timestamp，这个存储的是一个浮点数，并且是一个相对时间差，他记录的是从UTC1970年1月1日开始到某一个时刻的时间差值。

> The unix time stamp is a way to track time as a running total of seconds. This count starts at the Unix Epoch on January 1st, 1970 at UTC. Therefore, the unix time stamp is merely the number of seconds between a particular date and the Unix Epoch. It should also be pointed out (thanks to the comments from visitors to this site) that this point in time technically does not change no matter where you are located on the globe. This is very useful to computer systems for tracking and sorting dated information in dynamic and distributed applications both online and client side.
一般数据库中都存储utc标准时间或者unix时间戳，这样做的好处很明显，如果是全球化服务，1. 服务器可能部署在不同的时区，需要一个统一的时间来统一标准。2. 加上用户所在的时区即可知道用户所在地访问服务器的时间。

比如如果你有一个定时任务调度服务，他为全球用户提供cron定时任务的调度，而用户设置的都是按照他自己所在时区的调度时间，因此你的服务器必须要知道用户所在的时区，否则同样的设置，比如 硅谷用户设置的周一到周五每天的21,22,23点和北京的用户设置的周一到周五21,22,23点，假设他们设置的请求时间都是`UTC 2017-12-07 13:50:40.556828 周四`, 如果你的服务是按照服务器所在的时区的当地时间运行，那可能就会产生错误的调度行为，因此服务器的调度运行最好统一采用UNIX UTC时间，然后你调度之前统一将用户设置的时间利用时区信息转换到UTC标准时间，就可以准确调度了。这里调度器一般会计算出调度时刻到生效时刻的时间差，那就知道多久之后(时间差)需要调度用户任务。

## unaware 和 aware时间
unaware 时间这个时间本身并不带时区timezone的属性，例如 `2017-12-07 21:50:40.556828` 是一个不带时区的时间，那么这个时间可以放在任意一个时区，也就是说离开时区他是没有意义的，你无法判断他是哪个地方的时间点。

aware 时间本身会带省时区timezone属性，例如`2017-12-07 21:50:40.556888+08:00` 这个就表明自己是东八区的`2017-12-07 21:50:40.556888`, 这样你就知道现在世界上其他地方是什么时间，也就是说你知道了从UTC1970.1.1开始到这个时刻的绝对时间差，世界各地从UTC到这个时刻的时间差都是一样的，只是时间显示在不同的时区会不一样而已。

## date & time & datetime & calendar
python 中datetime模块有date time 以及datetime 和 timedelta几个类。分别表示日期，时间，日期+时间，时差等，详情可以参看文档。

以下是 unaware 和 aware 时间的一些转换，
```python
import time
import datetime
import pytz
from tzlocal import get_localzone
tz = get_localzone()
'''
following time have no time zone info, the time without time zone info means nothing for users in different time zone.
it does not denote that it is for which time zone.
it means that you see the time where you are, 

'''
# this is just for utc time zone time, but when your program in other time zone running, the now time is wrong for your user
unaware_utc_time = datetime.datetime.utcnow()
# this is the time without timezone info which manully generates.
unaware_time = datetime.datetime(2017,8,15,20,12,1,10)
# this is the time without timezone info but it is the time where your program runing in.
unaware_local_time = datetime.datetime.now()

'''
following time have time zone info. it denotes that its time is for the timezone itself.
'''
aware_local_time = datetime.datetime.now(tz)
aware_toronto_time = datetime.datetime(2017,8,15,20,12,1,10, pytz.timezone("America/Toronto"))
aware_utc_time = datetime.datetime.now(pytz.UTC)
replaced_timezone_aware = aware_toronto_time.replace(tzinfo=pytz.UTC)

from prettytable import PrettyTable
row = PrettyTable()
row.field_names = ['type', 'str', 'utc_offset', 'offset_seconds', 'unix_timestamp']
# 由于 python 3.3+ 才有 datetime.datetime.timestamp() 函数，因此我们使用 time_diff = (dt - datetime.datetime(1970,1,1)); time_diff.total_seconds()
# python 2.7+ 才有datetime.timedelta.total_seconds() 函数，如果是python 2.6 则使用 
'''
def timedelta_total_seconds(timedelta):
    return (
        timedelta.microseconds + 0.0 +
        (timedelta.seconds + timedelta.days * 24 * 3600) * 10 ** 6) / 10 ** 6
'''

def timestamp(dt):
    # we need keep +/- operands consistent in offset-naive, or offset-aware
    if dt.tzinfo:
        dt = dt.replace(tzinfo=None) # for aware, remove tzinfo transform to naive
    unix_epoch = datetime.datetime(1970,1,1) # unaware
    return (dt - unix_epoch).total_seconds()

row.add_row(['unaware_utc_time:', unaware_utc_time, unaware_utc_time.utcoffset(), None, timestamp(unaware_utc_time)])
row.add_row(['unaware_time:', unaware_time, unaware_time.utcoffset(), None, timestamp(unaware_time)])
row.add_row(('unaware_local_time:', unaware_local_time, unaware_local_time.utcoffset(), None, timestamp(unaware_local_time)))

row.add_row(['aware_local_time:', aware_local_time, aware_local_time.utcoffset(), aware_local_time.utcoffset().total_seconds(), timestamp(aware_local_time)])
row.add_row(('aware_toronto_time:', aware_toronto_time, aware_toronto_time.utcoffset(), aware_toronto_time.utcoffset().total_seconds(), timestamp(aware_toronto_time)))
row.add_row(('aware_utc_time:', aware_utc_time, aware_utc_time.utcoffset(), aware_utc_time.utcoffset().total_seconds(), timestamp(aware_utc_time)))
row.add_row(('replaced_timezone_aware:', replaced_timezone_aware, replaced_timezone_aware.utcoffset(), replaced_timezone_aware.utcoffset().total_seconds(), timestamp(replaced_timezone_aware)))
print row

print '{:25} {:50}'.format('unaware_utc_time:', repr(unaware_utc_time))
print '{:25} {:50}'.format('unaware_time:', repr(unaware_time))
print '{:25} {:50}'.format('unaware_local_time:', repr(unaware_local_time))
print '{:25} {:50}'.format('aware_local_time:', repr(aware_local_time))
print '{:25} {:50}'.format('aware_toronto_time:', repr(aware_toronto_time))
print '{:25} {:50}'.format('aware_utc_time:', repr(aware_utc_time))
print '{:25} {:50}'.format('replaced_timezone_aware:', repr(replaced_timezone_aware))

'''
+--------------------------+----------------------------------+------------------+----------------+----------------+
|           type           |               str                |    utc_offset    | offset_seconds | unix_timestamp |
+--------------------------+----------------------------------+------------------+----------------+----------------+
|    unaware_utc_time:     |    2017-12-07 14:44:06.333475    |       None       |      None      | 1512657846.33  |
|      unaware_time:       |    2017-08-15 20:12:01.000010    |       None       |      None      |  1502827921.0  |
|   unaware_local_time:    |    2017-12-07 22:44:06.333582    |       None       |      None      | 1512686646.33  |
|    aware_local_time:     | 2017-12-07 22:44:06.333641+08:00 |     8:00:00      |    28800.0     | 1512686646.33  |
|   aware_toronto_time:    | 2017-08-15 20:12:01.000010-05:18 | -1 day, 18:42:00 |    -19080.0    |  1502827921.0  |
|     aware_utc_time:      | 2017-12-07 14:44:06.333776+00:00 |     0:00:00      |      0.0       | 1512657846.33  |
| replaced_timezone_aware: | 2017-08-15 20:12:01.000010+00:00 |     0:00:00      |      0.0       |  1502827921.0  |
+--------------------------+----------------------------------+------------------+----------------+----------------+
unaware_utc_time:         datetime.datetime(2017, 12, 7, 14, 44, 6, 333475) 
unaware_time:             datetime.datetime(2017, 8, 15, 20, 12, 1, 10)     
unaware_local_time:       datetime.datetime(2017, 12, 7, 22, 44, 6, 333582) 
aware_local_time:         datetime.datetime(2017, 12, 7, 22, 44, 6, 333641, tzinfo=<DstTzInfo 'Asia/Shanghai' CST+8:00:00 STD>)
aware_toronto_time:       datetime.datetime(2017, 8, 15, 20, 12, 1, 10, tzinfo=<DstTzInfo 'America/Toronto' LMT-1 day, 18:42:00 STD>)
aware_utc_time:           datetime.datetime(2017, 12, 7, 14, 44, 6, 333776, tzinfo=<UTC>)
replaced_timezone_aware:  datetime.datetime(2017, 8, 15, 20, 12, 1, 10, tzinfo=<UTC>)
'''
```

### time 模块和 datetime 模块的简单转换
这里不管是 aware 还是 unaware 的datetime，转换都是直接拿出
```python
import time
unaware_time1 = datetime.datetime.utcfromtimestamp(
    time.mktime(aware_local_time.timetuple()))
print repr(unaware_time1)
print unaware_local_time.utctimetuple()
print time.mktime(unaware_local_time.utctimetuple())
print unaware_local_time.timetuple()
print time.mktime(unaware_local_time.timetuple())
print aware_local_time.utctimetuple()
print time.mktime(aware_local_time.utctimetuple())
print aware_local_time.timetuple()
print time.mktime(aware_local_time.timetuple())
print time.time()

# can't compare offset-naive and offset-aware datetimes
unaware_time == aware_time

'''
datetime.datetime(2017, 12, 7, 14, 44, 6)
time.struct_time(tm_year=2017, tm_mon=12, tm_mday=7, tm_hour=22, tm_min=44, tm_sec=6, tm_wday=3, tm_yday=341, tm_isdst=0)
1512657846.0
time.struct_time(tm_year=2017, tm_mon=12, tm_mday=7, tm_hour=22, tm_min=44, tm_sec=6, tm_wday=3, tm_yday=341, tm_isdst=-1)
1512657846.0
time.struct_time(tm_year=2017, tm_mon=12, tm_mday=7, tm_hour=14, tm_min=44, tm_sec=6, tm_wday=3, tm_yday=341, tm_isdst=0)
1512629046.0
time.struct_time(tm_year=2017, tm_mon=12, tm_mday=7, tm_hour=22, tm_min=44, tm_sec=6, tm_wday=3, tm_yday=341, tm_isdst=0)
1512657846.0
1512659346.58
'''
```
### 格式化输出时间
```python
import datetime

datetime.datetime.fromtimestamp(1509602220)
import time
now = time.strptime("2017-10-25T12:36:18Z", "%Y-%m-%dT%H:%M:%SZ")
dt = datetime.datetime.fromtimestamp(time.mktime(now))
dt
```

### python time 转换 cheetsheet



### 计算快照过期时间的小例子
```python
import datetime
import time
def timedelta_total_seconds(timedelta):
    return (
        timedelta.microseconds + 0.0 +
        (timedelta.seconds + timedelta.days * 24 * 3600) * 10 ** 6) / 10 ** 6

def timedelta(snapshot):
    now = datetime.datetime.utcnow()
    expire = datetime.timedelta(snapshot.expired_days) + snapshot.created_at
    delta = expire - now
    return delta
    

class Snapshot(object):
    def __init__(self, expired_days, created_at):
        self.expired_days = expired_days
        self.created_at = created_at
    def __str__(self):
        now = datetime.datetime.utcnow()
        expire = datetime.timedelta(snapshot.expired_days) + snapshot.created_at
        delta = timedelta_total_seconds(expire - now)
        return 'created_at: ' + str(self.created_at) + ', '\
                + 'expired_days: ' + str(self.expired_days) + ', ' \
                + 'expire: ' + str(expire) \
                + ', ' + 'wait: ' + str(delta)

snapshots = []
for i in range(10):
    snapshots.append(Snapshot(i, datetime.datetime.utcnow()))
    time.sleep(1)
    
snapshot = Snapshot(1, datetime.datetime.utcnow())    
now = datetime.datetime.utcnow()

delta = (datetime.timedelta(snapshot.expired_days) + snapshot.created_at) - now
timedelta_total_seconds(delta)

for snapshot in snapshots:
    print snapshot
    
rets = sorted(snapshots, key=timedelta, reverse=False)
for snapshot in rets:
    print snapshot
```