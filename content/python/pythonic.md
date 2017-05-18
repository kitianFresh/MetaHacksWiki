---
title: "pythonic"
date: 2017-04-23 22:19
---

# Python @staticmethod 和 @classmethod


```python
class Person(object):
    def __init__(self, age):
        self.age = age
    
    def get_age(self):
        return self.age
```


```python
Person.get_age
```




    <unbound method Person.get_age>



显示的是未绑定的方法, 为什么? 因为 Python 对象方法需要绑定到某一个实例才可以使用


```python
Person.get_age()
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-46-2e11244cec48> in <module>()
    ----> 1 Person.get_age()
    

    TypeError: unbound method get_age() must be called with Person instance as first argument (got nothing instead)



```python
Person.get_age(Person(23))
```




    23



这里其实就相当于 `Person(23)`　这个对象和我们的　`Person.get_age`　方法绑定了．这也是为什么实例方法都需要 `self` 做参数的原因了；

但是，　如果我们不记得类名字了，　就无法这样调用方法了．　因此　Python 提供了　Syntax Sugar, 直接通过和　Java 一样的方式调用绑定的实例方法即可！


```python
Person(23).get_age()# <==> Person.get_age(Person(23))
```




    23




```python
Person(22).get_age
```




    <bound method Person.get_age of <__main__.Person object at 0x7f5b24107e10>>



每一个绑定的方法，都可以获取他所绑定到的对象，　使用　方法　的　`__self__` 属性即可；


```python
m = Person(23).get_age
m.__self__
```




    <__main__.Person at 0x7f5b24107d50>




```python
m == m.__self__.get_age
```




    True



In Python 3, the functions attached to a class are *not* considered as *unbound method* anymore, but as simple functions, that are bound to an object if required. So the principle stays the same, the model is just simplified.

```python
>>> class Pizza(object):
...     def __init__(self, size):
...         self.size = size
...     def get_size(self):
...         return self.size
...
>>> Pizza.get_size
<function Pizza.get_size at 0x7f307f984dd0>
```

## Static Methods
对于属于一个类，但是却从不使用该类实例变量的方法来说，　使用静态方法就够了．**他既不和类的实例(self)绑定，也不和类本身(cls)绑定．只是意义上属于类的一个普通方法而已．**


```python
class Pizza(object):
    @staticmethod
    def mix_ingredients(x, y):
        return x + y
    
    def cook(self):
        return self.mix_ingredients(self.cheese, self.vegetables)
```

1. **Python 会将每一个实例方法和一个实例化对象绑定起来，而且绑定过的实例方法在Python 中也是对象，即使是同一个方法，每一个新对象都要生成一个新的绑定方法对象，非常耗费空间和时间，而使用　staticmethod 不会进行对象绑定；**

2. **@staticmethod 支持子类重写，即多态．如果你使用一个模块内的顶层函数　mix_ingredients, 继承自　Pizza　的子类并不能改变行为如果你不重写cook方法．**


```python
print Pizza().cook is Pizza().cook
print Pizza().mix_ingredients is Pizza().mix_ingredients
print Pizza().mix_ingredients is Pizza.mix_ingredients
```

    False
    True
    True


## Class Methods
**并不和对象绑定，但是和类绑定的方法, 需要加上　`cls` 参数**


```python
class Pizza(object):
    radius = 23
    @classmethod
    def get_radius(cls):
        return cls.radius
```


```python
Pizza.get_radius
```




    <bound method type.get_radius of <class '__main__.Pizza'>>




```python
Pizza().get_radius
```




    <bound method type.get_radius of <class '__main__.Pizza'>>




```python
Pizza.get_radius.__self__ is Pizza().get_radius.__self__
```




    True




```python
Pizza.get_radius == Pizza().get_radius
```




    True




```python
Pizza.get_radius()
```




    23



classmethod 的用法

1. **FactoryMethods. 如果我们需要使用一些预处理来创建实例，　使用 @staticmethod 必须硬编码类名　Pizza 在我们的函数中，　这样任何集成　Pizza 的子类就不能够使用工厂方法为自己所用了．**

```Python
class Pizza(object):
    def __init__(self, ingredients):
        self.ingredients = ingredients
 
    @classmethod
    def from_fridge(cls, fridge):
        return cls(fridge.get_cheese() + fridge.get_vegetables())
```
2. **需要调用静态方法的静态方法直接使用　classmethod. 原因还是因为在静态方法中又必须硬编码类名，　子类就无法；**
```python
class Pizza(object):
    def __init__(self, radius, height):
        self.radius = radius
        self.height = height
 
    @staticmethod
    def compute_area(radius):
         return math.pi * (radius ** 2)
 
    @classmethod
    def compute_volume(cls, height, radius):
         return height * cls.compute_area(radius)
 
    def get_volume(self):
        return self.compute_volume(self.height, self.radius)
```

## Abstract Methods
定义在基类中还未被实现的方法．
```python
class Pizza(object):
    def get_radius(self):
        raise NotImplementedError
```

任何继承自　Pizza 的子类都必须覆盖　get_radius 方法，　否则就会在调用的时候抛出　异常．
```python
>>> Pizza()
<__main__.Pizza object at 0x7fb747353d90>
>>> Pizza().get_radius()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in get_radius
NotImplementedError
```

但是以上实现方式会使得异常在调用该方法的时候才会发生，　要想提前知道方法未实现，　可以使用　`abc`　模块．
```python
import abc
 
class BasePizza(object):
    __metaclass__  = abc.ABCMeta
 
    @abc.abstractmethod
    def get_radius(self):
         """Method that should do something."""
```


```python
class Methods:
  def i_method(self,x):
    print(self,x)

  def s_method(x):
    print(x)

  def c_method(cls,x):
    print(cls,x)

  s_method = staticmethod(s_method)
  c_method = classmethod(c_method)

obj = Methods()

obj.i_method(1)
Methods.i_method(obj, 2)

obj.s_method(3)
Methods.s_method(4)

obj.c_method(5)
Methods.c_method(6)
```

    (<__main__.Methods instance at 0x7f5b240c8e60>, 1)
    (<__main__.Methods instance at 0x7f5b240c8e60>, 2)
    3
    4
    (<class __main__.Methods at 0x7f5b24067db8>, 5)
    (<class __main__.Methods at 0x7f5b24067db8>, 6)


以上实现和一下实现等价：


```python
class Methods:
  def i_method(self,x):
    print(self,x)

  @staticmethod
  def s_method(x):
    print(x)

  @classmethod
  def c_method(cls,x):
    print(cls,x)

obj = Methods()

obj.i_method(1)
Methods.i_method(obj, 2)

obj.s_method(3)
Methods.s_method(4)

obj.c_method(5)
Methods.c_method(6)
```

    (<__main__.Methods instance at 0x7f5b24039200>, 1)
    (<__main__.Methods instance at 0x7f5b24039200>, 2)
    3
    4
    (<class __main__.Methods at 0x7f5b24067e88>, 5)
    (<class __main__.Methods at 0x7f5b24067e88>, 6)



```python
class S:
   nInstances = 0
   def __init__(self):
      S.nInstances = S.nInstances + 1

   @staticmethod
   def howManyInstances():
      print('Number of instances created: ', S.nInstances)

a = S()
b = S()
c = S()
S.howManyInstances()
```

    ('Number of instances created: ', 3)


## 参考
　- [guide-python-static-class-abstract-methods](https://julien.danjou.info/blog/2013/guide-python-static-class-abstract-methods)



# python dict按照key 排序：
1、method 1.

```python
items = dict.items()
items.sort()
for key,value in items:
   print key, value # print key,dict[key]
```
2、method 2.

```python
print key, dict[key] for key in sorted(dict.keys())
```
　

# python dict按照value排序：
method 1：

把dictionary中的元素分离出来放到一个list中，对list排序，从而间接实现对dictionary的排序。这个“元素”可以是key，value或者item。

method2：

#用lambda表达式来排序，更灵活：
```python
# sorted(iterable, cmp=None, key=None, reverse=False) --> new sorted list

sorted(dict.items(), lambda x, y: cmp(x[1], y[1]))
#降序
sorted(dict.items(), lambda x, y: cmp(x[1], y[1]), reverse=True)

print sorted(dict1.items(), key=lambda d: d[0])
print sorted(dict1.items(), key=lambda d: d[1])
```

# Python 调用 shell 命令行程序

## Python实现的文件编码转码工具

```python
#!/usr/bin/python
# -*- coding: utf-8 -*-

import glob  
import subprocess

#父目录中的.py文件  
files = glob.iglob(r'./*.csv')

for f in files:
    subprocess.call("enca -L zh_CN " + f, shell=True)

for f in files:
    subprocess.call("enca -L zh_CN -x utf-8 " + f, shell=True)
```

## Python 实现解释器命令行中自动补全
第一种方法是直接每次先执行以下语句:
```python
import readline, rlcompleter; readline.parse_and_bind("tab: complete")
```
方法二新建一个 `.pythonstartup.py` 的脚本, 然后在 `.bashrc` 中加入 `export PYTHONSTARTUP = ~/.pythonstartup.py` 
```python
#.pythonstartup.py
import readline, rlcompleter 
readline.parse_and_bind("tab: complete")
```
最后执行 `source .bashrc` 生效. 然后在解释器中使用 `tab` 就会是自动补全了, 不再是一个缩进!缩进只能使用四个空格;