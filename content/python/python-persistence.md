---
title: "python-persistence"
date: 2017-12-07 23:26
---

# python serialization 序列化
```python
import cPickle as pickle

def job():
    print 'hehe'
print repr(job)
'''
含有 __slots__ 的类对象将不再支持动态添加新成员，因为内存固定了。一般用于大规模少量属性的对象，可节省内存；
不含__slots__ 的类对象其实是通过一次词典来维护对象内部成员的，因此更耗费内存。
''' 
class Family(object):
    _resource_name_ = 'father'
    __slots__ = ['father', 'var1', 'func'] # 必须与self 的成员变量个数保持一致
    def __init__(self, father, func, var1 = 1):
        self.father, self.var1, self.func = father, var1, func
    def __getstate__(self):
        return {'father':self.father, 'var1':self.var1, 'func': self.func}
    def __setstate__(self, state):
        self.father = state['father']
        self.var1 = state['var1']
        self.func = state['func']
        
class Country(object):
    __slots__ = ['father', 'var1', 'func']
    def __init__(self, father, func, var1 = 1):
        self.father, self.var1, self.func = father, var1, func
    def __getstate__(self):
        return { slot: getattr(self, slot) for slot in Country.__slots__}
    def __setstate__(self, state):
        for slot in state:
            setattr(self, slot, state[slot])

foo = Family('father', job, 1)
foo.var1 = 2
foo_pickled = pickle.dumps(foo.__getstate__())
foo2 = pickle.loads(foo_pickled)
print(repr(foo2))
# print(foo2.var1) # AttributeError: 'dict' object has no attribute 'var1'
family = Family.__new__(Family)
family.__setstate__(foo2)

print(foo2['var1'])
print(repr(family))
print(family.var1)
print repr(family.func)

coo = Country('father', job, 2)
c = pickle.loads(pickle.dumps(coo))
assert c.father == 'father'
assert c.func == job
assert c.var1 == 2

class T():
    def __init__(self):
        self.val = 1

t = T()
tt = pickle.loads(pickle.dumps(t))
assert tt.val == 1
tt.var2 = 2222
tt.job = job
print tt.var2
tt.job()
family.mother = 'mother'
```

```python
import cPickle as pickle

class Father(object):
    def __init__(self, name):
        self.name = name
        
    def __call__(self, *args, **kwargs):
        print 'Father was called by %s %s' % (args, kwargs)

class Family(object):
    __slots__ = ['father', 'var1']
    def __init__(self, father, var1 = 1):
        self.father, self.var1 = father, var1
    def __getstate__(self):
        return self.father, self.var1
    def __setstate__(self, state):
        self.father, self.var1 = state

father = Father('tian')
foo = Family(father,1)
foo.var1 = 2
foo_pickled = pickle.dumps(foo)
foo2 = pickle.loads(foo_pickled)
print(repr(foo2))
# <__main__.Family object at 0xb77037ec>
print foo2.father
print(foo2.var1)
# 2
dir(Family)

Family.__dict__
for key in Family.__dict__.items():
    print key
```


# python json