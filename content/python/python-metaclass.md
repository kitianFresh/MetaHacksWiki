---
title: "python-metaclass"
date: 2017-12-07 23:37
---



```python
from functools import wraps, partial
import logging
def debug(func=None, prefix=''):
    if func is None:
        return partial(debug, prefix=prefix)
    log = logging.getLogger(func.__module__)
    msg = prefix + func.__name__
    @wraps(func)
    def wrapper(*args, **kwargs):
        log.debug(msg)
        print(msg)
        return func(*args, **kwargs)
    return wrapper
```


```python
@debug(prefix='***')
def add(x, y):
    return x + y

add(1,3)
```

    ***add





    4



Debug一个类的所有实例方法


```python
def debugmethods(cls):
    for name, val in vars(cls).items():
        if callable(val):
            print name, val
            setattr(cls, name, debug(val))
            
    return cls
```


```python
@debugmethods
class Spam:
    def grok(self):
        pass
    
    def bar(self):
        pass
    
    def foo(self):
        pass
    
    @classmethod
    def gun(cls):
        pass
    
    @staticmethod
    def fire():
        pass
```

    grok <function grok at 0x10ed6c050>
    bar <function bar at 0x10ed6c0c8>
    foo <function foo at 0x10ed6c140>



```python
vars(Spam).items() # Spam.__dict__.items()
```




    [('grok', <function __main__.grok>),
     ('__module__', '__main__'),
     ('bar', <function __main__.bar>),
     ('fire', <staticmethod at 0x10ece4b78>),
     ('gun', <classmethod at 0x10ece4980>),
     ('foo', <function __main__.foo>),
     ('__doc__', None)]



以上缺点就是并不能装饰classmethod和staticmethod，为什么呢？从打印来看，你会发现函数地址不在一个scope下


```python
def debugattr(cls):
    orig_getattribute = cls.__getattribute__ 
    def __getattribute__(self, name):
        print('Get:', name)
        return orig_getattribute(self, name)
    
    cls.__getattribute__ = __getattribute__
    
    return cls
```


```python
@debugattr
class Point(object):#Python2.7中一定要继承object，否则没有object的一些属性
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def __getattr__(self, e):
        return self.__dict__[e]
        
        
p = Point(2,3)
p.x
dir(Point)
```

    ('Get:', 'x')





    ['__class__',
     '__delattr__',
     '__dict__',
     '__doc__',
     '__format__',
     '__getattr__',
     '__getattribute__',
     '__hash__',
     '__init__',
     '__module__',
     '__new__',
     '__reduce__',
     '__reduce_ex__',
     '__repr__',
     '__setattr__',
     '__sizeof__',
     '__str__',
     '__subclasshook__',
     '__weakref__']



解决办法metaclass


```python
class debugmeta(type):
    def __new__(cls, clsname, bases, clsdict):
        clsobj = super(debugmeta, cls).__new__(cls, clsname, bases, clsdict)
        clsobj = debugmethods(clsobj)
        return clsobj

'''
# python 3 syntax
class Base(metaclass=debugmeta):
    pass
'''
class Base(object):
    __metaclass__ = debugmeta
```

    __metaclass__ <class '__main__.debugmeta'>



```python
isinstance(Point, type)
dir(debugmeta)
```




    ['__abstractmethods__',
     '__base__',
     '__bases__',
     '__basicsize__',
     '__call__',
     '__class__',
     '__delattr__',
     '__dict__',
     '__dictoffset__',
     '__doc__',
     '__eq__',
     '__flags__',
     '__format__',
     '__ge__',
     '__getattribute__',
     '__gt__',
     '__hash__',
     '__init__',
     '__instancecheck__',
     '__itemsize__',
     '__le__',
     '__lt__',
     '__module__',
     '__mro__',
     '__name__',
     '__ne__',
     '__new__',
     '__reduce__',
     '__reduce_ex__',
     '__repr__',
     '__setattr__',
     '__sizeof__',
     '__str__',
     '__subclasscheck__',
     '__subclasses__',
     '__subclasshook__',
     '__weakrefoffset__',
     'mro']




```python
class Spam(Base):
    def __init__(self, name):
        self.name = name
    def bar(self):
        print "I'm Spam.bar"
```

    bar <function bar at 0x1083532a8>
    __init__ <function __init__ at 0x1083539b0>



```python
dir(Spam)
```




    ['__class__',
     '__delattr__',
     '__dict__',
     '__doc__',
     '__format__',
     '__getattribute__',
     '__hash__',
     '__init__',
     '__metaclass__',
     '__module__',
     '__new__',
     '__reduce__',
     '__reduce_ex__',
     '__repr__',
     '__setattr__',
     '__sizeof__',
     '__str__',
     '__subclasshook__',
     '__weakref__',
     'bar']




```python
body = '''
    def __init__(self, name):
        self.name = name
    def bar(self):
        print "I'm Spam.bar"
'''
```


```python
clsdict = type.__prepare__('Spam', (Base,))
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-108-cf797e00aba3> in <module>()
    ----> 1 clsdict = type.__prepare__('Spam', (Base,))
    

    AttributeError: type object 'type' has no attribute '__prepare__'


注意，Python 2 中， type 并未实现 `__prepare__()` 方法。在 Python 3 中， type 实现了 `__prepare__()` 方法。 Python 解释器构造类的时候，在解析class body 之前，会调用`__prepare__()`函数。并且Python 解释器在调用 `__prepare__()` 之前总是先检查`__prepare__()` 的存在。 存在就会调用，不存在就返回一个一般的dict。
```python
def prepare_class(name, *bases, metaclass=None, **kwargs):
    if metaclass is None:
        metaclass = compute_default_metaclass(bases)
    prepare = getattr(metaclass, '__prepare__', None)
    if prepare is not None:
        return prepare(name, bases, **kwargs)
    else:
        return dict()
```

>__prepare__ returns a dictionary-like object which is used to store the class member definitions during evaluation of the class body. In other words, the class body is evaluated as a function block (just like it is now), except that the local variables dictionary is replaced by the dictionary returned from __prepare__. This dictionary object can be a regular dictionary or a custom mapping type.

`__prepare__` 函数返回一个类词典对象用来存储在解析class body 期间发现的类成员定义。局部变量词典会被该方法返回的词典替换。参考[PEP 3115 -- Metaclasses in Python 3000](https://www.python.org/dev/peps/pep-3115/?cm_mc_uid=16088229483114992455164&cm_mc_sid_50200000=1503827755)


```python
class MyInt(int):
    def __add__(self, other):
        print "I am a new int:)"
        return super(MyInt, self).__add__(other)
    
i = MyInt(2)
print i+3
```

    I am a new int:)
    5



```python
type(i), type(MyInt),type(tuple), type(list), type(dict), type(int), type(type)
```




    (__main__.MyInt, type, type, type, type, type, type)




```python
dir(type)
```




    ['__abstractmethods__',
     '__base__',
     '__bases__',
     '__basicsize__',
     '__call__',
     '__class__',
     '__delattr__',
     '__dict__',
     '__dictoffset__',
     '__doc__',
     '__eq__',
     '__flags__',
     '__format__',
     '__ge__',
     '__getattribute__',
     '__gt__',
     '__hash__',
     '__init__',
     '__instancecheck__',
     '__itemsize__',
     '__le__',
     '__lt__',
     '__module__',
     '__mro__',
     '__name__',
     '__ne__',
     '__new__',
     '__reduce__',
     '__reduce_ex__',
     '__repr__',
     '__setattr__',
     '__sizeof__',
     '__str__',
     '__subclasscheck__',
     '__subclasses__',
     '__subclasshook__',
     '__weakrefoffset__',
     'mro']




```python
class InterfaceMeta(type):
    def __new__(cls, name, parents, dct):
        # create a class_id if it's not specified
        if 'class_id' not in dct:
            dct['class_id'] = name.lower()
        
        # open the specified file for writing
        if 'file' in dct:
            filename = dct['file']
            dct['file'] = open(filename, 'w')
        
        # we need to call type.__new__ to complete the initialization
        return super(InterfaceMeta, cls).__new__(cls, name, parents, dct)
```


```python
Interface = InterfaceMeta('Interface', (), dict(file='tmp.txt'))

print(Interface.class_id)
print(Interface.file)

```

    interface
    <open file 'tmp.txt', mode 'w' at 0x10ed49150>


每次都这样构造出一个新类会非常心累，因此和 装饰器一样 Python 又为我们提供了 语法糖。定义一个 有`__metaclass__` 属性的类，该类的构造自动会使用该 `__metaclass__` 指向的元类来构造。简化和方便了我们写代码；


```python
class Interface(object):
    __metaclass__ = InterfaceMeta
    file = 'tmp.txt'
    
print(Interface.class_id)
print(Interface.file)

```

    interface
    <open file 'tmp.txt', mode 'w' at 0x10ed49270>


继承的子类也会只要元类未被修改，都会通过该元类来构造出来。


```python
class UserInterface(Interface):
    file = 'foo.txt'
    
print(UserInterface.file)
print(UserInterface.class_id)
```

    <open file 'foo.txt', mode 'w' at 0x10ed490c0>
    userinterface


# Abstract Base Class(ABC)
抽象基类ABC，主要定义了基本类和最基本的抽象方法，为子类提供共有API。这个abc 包下有一个 abc.ABCMeta, 我们已经知道了元类，又知道了ABC是抽象基类，那这个 ABCMeta 就是创建 ABC 抽象基类的元类了。抽象基类一般不实现具体方法。都是定义的抽象方法和属性: `@abstractmethod` 和 `@abstarctproperty`.

抽象基类当然是用来继承的，这里的继承分两种：直接代码上的继承和将其他类注册为虚拟子类的继承方式。区别就是，直接继承，必须要实现抽象方法；而通过注册(register) 是可以不实现抽象方法的，但是还是会被认为是该ABC的子类，可以使用 issubclass 和 issubinstance 验证。


```python
from abc import ABCMeta

class MyABC():
    __metaclass__ = ABCMeta
    pass

MyABC.register(tuple)

assert issubclass(tuple, MyABC)
assert isinstance((), MyABC)
```

以下是 issubclass 和 isinstance 的真正调用函数 `__instancecheck__` 和 `__subclasscheck__`
```python
class ABCMeta(type):

    def __instancecheck__(cls, inst):
        """Implement isinstance(inst, cls)."""
        return any(cls.__subclasscheck__(c)
                   for c in {type(inst), inst.__class__})

    def __subclasscheck__(cls, sub):
        """Implement issubclass(sub, cls)."""
        candidates = cls.__dict__.get("__subclass__", set()) | {cls}
        return any(c in candidates for c in sub.mro())
```
更多详细内容可以参考[PEP 3119](https://www.python.org/dev/peps/pep-3119/?cm_mc_uid=16088229483114992455164&cm_mc_sid_50200000=1503827755)

# six（python 2 * 3 = six）
six 其实就是一个 Python 2 和 Python 3 的使用元类写法的兼容库。Python 2 中元类的使用通过 `__metaclass__`, Python 3 中是通过参数的形式 Class(Base1, Base2, metaclass=ABCMeta) 完成的。

装饰器是给一个类添加元类的23兼容写法。`@six.add_metaclass(metaclass)`

```python
@add_metaclass(Meta)
class MyClass(object):
    pass
That code produces a class equivalent to

class MyClass(object, metaclass=Meta):
    pass
on Python 3 or

class MyClass(object):
    __metaclass__ = MyMeta
on Python 2.

Note that class decorators require Python 2.6. 

However, the effect of the decorator can be emulated on Python 2.5 like so:

class MyClass(object):
    pass
MyClass = add_metaclass(Meta)(MyClass)

```


```python

```
