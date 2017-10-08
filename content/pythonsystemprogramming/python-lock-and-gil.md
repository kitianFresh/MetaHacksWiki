---
title: "python-lock-and-GIL"
date: 2017-09-29 19:58
---

# Python thread lock
>为什么 python thread 有全局的 GIL 锁，python 多线程并发还需要枷锁？

首先，锁的目的是什么， 是保证事务的原子性！ 这个事务的概念可以很大，也可以很小，大到一些列复杂操作组成的一个业务单元，小到执行一个计数加1操作。 因此， 原子性是根据业务需求来定的！原子性的范围决定了临界区的范围，也导致了锁开始和结束的位置我们称之为锁的粒度。

Python 没有真正的多线程，这句话一半对一半不对。Python不是没有多线程，Python 有多线程，只是多线程无法充分利用cpu并行（由于GIL的存在）；那这个 GIL 既然是个锁，他锁的粒度有多大，我们说Python解释器在一个时刻只可能执行一个线程，这是正确的，因为GIL锁住的是线程的字节指令序列，其他线程要执行，必须拿到GIL才能执行，因此Python不存在真正意义上的多线程完全并行。实际上，永远是穿行的执行线程。那么这是不是意味着，多线程的对全局计数器的加1操作就不用枷锁呢？

```python
import dis
num = 0
def f():
    global num
    num += 1

import threading
ts = []
for i in range(1000000):
    t = threading.Thread(target=f)
    ts.append(t)
    t.start()

for t in ts:
    t.join()

print num
```
这个的输出结果预期应该是 999999， 但结果却不是，我试验是999987. 我们dis出f的字节码看看；
```python
  5           0 LOAD_GLOBAL              0 (num)
              3 LOAD_CONST               1 (1)
              6 INPLACE_ADD         
              7 STORE_GLOBAL             0 (num)
             10 LOAD_CONST               0 (None)
             13 RETURN_VALUE        
```

```python
class Manager():
    count = 0
    def __init__(self):
        self.count += 1
    @classmethod
    def get_count(cls):
        return cls.count

manager = None
def get_default_manager():
    global manager
    if manager is None:
        manager = Manager()
    return manager

import dis
dis.dis(get_default_manager)

import threading
ts = []
for i in range(10000000):
    t = threading.Thread(target=get_default_manager)
    ts.append(t)
    t.start()

for t in ts:
    t.join()

```

```python
12           0 LOAD_GLOBAL              0 (manager)
              3 LOAD_CONST               0 (None)
              6 COMPARE_OP               8 (is)
              9 POP_JUMP_IF_FALSE       24

 13          12 LOAD_GLOBAL              2 (Manager)
             15 CALL_FUNCTION            0
             18 STORE_GLOBAL             0 (manager)
             21 JUMP_FORWARD             0 (to 24)

 14     >>   24 LOAD_GLOBAL              0 (manager)
             27 RETURN_VALUE 
```