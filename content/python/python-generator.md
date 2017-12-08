---
title: "python-generator"
date: 2017-12-07 23:37
---




# iterator
迭代器在Python中非常常见，在增强for loop 中，其实就是隐式的在使用迭代器，for loop通常会被转化为基本的while loop从而使用迭代器。
```python
for index(es) in iterable:  # indexes for unpacked assignment: e.g, key,value
      block-body
  [else:
      block-else]
```
其实被翻译成了更原始的while loop.
```python
_hidden = iter(iterable)		# ultimately calls iterable.__iter__()
try:
    while True:
        index(es) = next(_hidden)	# ultimatelyl calls _hidden.__next__()
        block-body
except StopIteration:
    pass				# A place-holder, when [] is discarded;
    [block-else]
```
其中， iter() 函数和 next() 函数一样，都调用对象内部的 \_\_iter\_\_() 和 \_\_next\_\_().
```
def iter(i):
    return i.__iter__()

def next(i):
    return i.__next__()
```


```python
def abs_diff1(iterable):
    alist = list(iterable)#有些iterable不支持切片（eg. tuple-comprehension）,因此这里先做转换
    return [abs(a-b) for a, b in zip(alist, alist[1:])]

def abs_diff2(iterable):
    result = []
    i = iter(iterable)
    v2 = next(i)
    try:
        while True:
            v1, v2 = v2, next(i)
            result.append(abs(v1-v2))
    except StopIteration:
        pass
    return result
alist = range(10, 1, -2)
%time abs_diff1(alist)
#%time abs_diff2(alist)

```

    CPU times: user 6 µs, sys: 2 µs, total: 8 µs
    Wall time: 10 µs





    [2, 2, 2, 2]




```python
def count(n):
    i = 0
    while i < n:
        yield i
        i += 1
for i in count(10):
    print i,
```

    0 1 2 3 4 5 6 7 8 9



```python
for i in range(1,6):
    print(i)
    if i == 4: break
else:
    print('exec else')

#v1
_hidden = iter(range(1,6))
try:
    while True:
        i = next(_hidden)
        print(i)
        if i == 4: break
except StopIteration:
    
    print('exec else')
    
#v2
_hidden = iter(range(1,6))
while True:
    try:
        i = next(_hidden)
        print(i)
        if i == 4: break
    except StopIteration:
        print('exec else')
        break
```

    1
    2
    3
    4
    1
    2
    3
    4
    1
    2
    3
    4


## 自定义迭代器Iterator
Python 2 实现 \_\_iter\_\_() 和 next() 函数，就是 Iterables 对象。Iteration Protocol. Python 3版本next被更改成了\_\_next\_\_()方法。


```python
class countdown(object):
    def __init__(self, start):
        self.count = start
    def __iter__(self):
        return self
    def next(self):
        if self.count < 0:
            raise StopIteration
        r = self.count
        self.count -= 1
        return r
    
class Countdown:
    def __init__(self,last):
        self.last = last	# self.last never changes
    
    # __iter__ must return an object on which __next__ can be called; it returns
    # self, which is an object of the Countdown class, which defines __next__.
    # Later we will see a problem with returning self (when the same Countdown
    # object is iterated over simultaneously), and how to solve that problem. 

    def __iter__(self):
        self.n = self.last	# n attribute is added to the namespace here 
        return self             # (not in __init__) and processed in __next__
    
    def next(self):
        if self.n < 0:
            raise StopIteration
        else:
            answer = self.n	# or, without the temporary, but more confusing
            self.n -= 1		#  self.n -= 1
            return answer       #  return self.n+1
        
for i in countdown(5):
    print i,
    
print
for a in countdown(2):
    for b in countdown(2):
        print(a,b)

c = countdown(2)
# c = Countdown(2)
for a in c:
    for b in c:
        print(a, b)

print('--------------')

r = range(2,-1,-1)
for a in r:
    for b in r:
        print(a, b)

print('--------------')

c = countdown(2)
# c = Countdown(2)
_hidden_1 = iter(c)
try:
    while True:
        a = next(_hidden_1)
        _hidden_2 = iter(c)
        try:
            while True:
                b = next(_hidden_2)
                print(a, b)
        except StopIteration:
            print('inner exec')
except StopIteration:
    print('outer exec')
```

    5 4 3 2 1 0
    (2, 2)
    (2, 1)
    (2, 0)
    (1, 2)
    (1, 1)
    (1, 0)
    (0, 2)
    (0, 1)
    (0, 0)
    (2, 1)
    (2, 0)
    --------------
    (2, 2)
    (2, 1)
    (2, 0)
    (1, 2)
    (1, 1)
    (1, 0)
    (0, 2)
    (0, 1)
    (0, 0)
    --------------
    (2, 1)
    (2, 0)
    inner exec
    outer exec


可以看出，以上的 countdown 是有问题的，我们必须让每次 for loop 都从新的 iterator 开始，即每个for loop 中的 iterator有其自己的状态，相互之间不干扰，我们方法就是在外部类的\_\_iter\_\_()方法中定义一个local nested class。然后在这个内部类中只用实现 \_\_init\_\_() 和 next() 方法即可。外部类就不用next了，代码直接放入内部类的next()代码中，这样每一次 iter()的调用，就会返回一个新对象，状态互不干扰。

>\_\_iter\_\_ method defining a LOCAL or NESTED CLASS (these classes define only
an \_\_init\_\_ and\_\_next\_\_ methods) and returning a new object constructed from
this local/nested class every time \_\_iter\_\_ is called.

- [Iterators via Classes
](https://www.ics.uci.edu/~pattis/ICS-33/lectures/iteratorclasses.txt)


```python
class NewCountdown(object):
    def __init__(self, last):
        self.last = last
    def __iter__(self):
        class CD_iter:
            def __init__(self, last):
                self.count = last
            def next(self):
                if self.count < 0:
                    raise StopIteration
                r = self.count
                self.count -= 1
                return r
        return CD_iter(self.last)

c = NewCountdown(2)
for a in c:
    for b in c:
        print(a,b)
```

    (2, 2)
    (2, 1)
    (2, 0)
    (1, 2)
    (1, 1)
    (1, 0)
    (0, 2)
    (0, 1)
    (0, 0)



```python

```

# Generator
用于一次性操作，list可以多次遍历，但是要想再一次遍历已经运行完的generator必须重新生成该generator。generator 是 iterable 的，因此可以当 iterable 使用。但是他并不是一个iterator对象，而是generator对象。
- [pydoc](https://docs.python.org/2/reference/expressions.html#generator.send)
- [generators](https://www.ics.uci.edu/~pattis/ICS-33/lectures/generators.txt)


```python
a = [1,2,3,4]
iterator = [2*x for x in a]
iterator
```




    [2, 4, 6, 8]




```python
generator = (2*x for x in a)
generator
```




    <generator object <genexpr> at 0x10b4c3dc0>



- General syntax
```python
(expression for i in s if condition)
```

- What it means
```python
 for i in s:
     if condition:
         yield expression
 ```

>A yield-expression must always be parenthesized except when it occurs at the top-level expression on the right-hand side of an assignment. So
[PEP342](https://www.python.org/dev/peps/pep-0342/)

```python
#legal
x = yield 42
x = yield
x = 12 + (yield 42)
x = 12 + (yield)
foo(yield 42)
foo(yield)
# illegal
x = 12 + yield 42
x = 12 + yield
foo(yield 42, 12)
foo(yield, 12)
```
要想是一个对象或函数成为generator，只要在函数体中使用`yield`关键字就可以了.


```python
def echo(value=None):
    print "Execution starts when 'next()' is called for the first time."
    try:
        while True:
            try:
                value = (yield value)
            except Exception, e:
                value = e
    finally:
        print "Don't forget to clean up when 'close()' is called."
```


```python
import math
def is_prime(number):
    if number <= 1:
        return False
    for current in range(2, int(math.sqrt(number))+1):
        if number % current == 0:
            return False
    return True
def get_primes(number):
    while True:
        if is_prime(number):
            number = yield number
        number += 1
def print_successive_primes(iterations, base=10):
    prime_generator = get_primes(base)
    prime_generator.send(None)
    for power in range(iterations):
        print(prime_generator.send(base ** power))

print_successive_primes(4)

def primes(max=None):
    p = 2
    while max == None or p <= max:
        if is_prime(p):
            yield p
        p += 1

pg = primes(10)
print(next(pg))
print(next(pg))
print(next(pg))
print(next(pg))
```

    2
    11
    101
    1009
    2
    3
    5
    7



```python
from inspect import isfunction, isgenerator

print('primes', type(primes))
print('primes()', type(primes()))
print('isfunction(primes)', isfunction(primes))
print('isgenerator(primes())', isgenerator(primes()))
print(dir(primes()))
```

    ('primes', <type 'function'>)
    ('primes()', <type 'generator'>)
    ('isfunction(primes)', True)
    ('isgenerator(primes())', True)
    ['__class__', '__delattr__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__iter__', '__name__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'close', 'gi_code', 'gi_frame', 'gi_running', 'next', 'send', 'throw']


需要注意的是，primes 是一个函数对象，而 primes() 是一个 generator 对象。primes函数调用之后，由于内部含有`yield`关键字而被认为是一个generator函数，此时函数体并没有执行，而是返回一个generator对象!而后要使用generator执行他的代码，需要显式或隐式调用 `next()` 或者 `send()` 函数.

对于函数调用，调用后，执行并返回；
对于生成器调用，调用之后，执行到 `yield` 关键字之后中断进入断点状态(suspend)，等待下一次从断点恢复执行(resume).

generator有四个函数，next,send,throw,close

# generator application for system programming
## Python version of Unix 'tail -f'


```python
import time
def follow(thefile):
    thefile.seek(0,2) # 设置文件指针到文件末尾
    while True:
        line = thefile.readline()
        # 以下是模拟阻塞，因为一旦thefile.readline()读到文件末尾，就会返回None。
        # 为了模拟能够一直读到最新的尾部数据，必须不断的poll，这里每隔0.1s轮询一次。
        if not line:
            time.sleep(0.1)
            continue
        yield line
logfile = open("access-log")
for line in follow(logfile):
    print line,
logfile.close()
```

    pearl
    java
    python
    lua



    ---------------------------------------------------------------------------

    KeyboardInterrupt                         Traceback (most recent call last)

    <ipython-input-11-7b507f742f61> in <module>()
          9         yield line
         10 logfile = open("access-log")
    ---> 11 for line in follow(logfile):
         12     print line,
         13 logfile.close()


    <ipython-input-11-7b507f742f61> in follow(thefile)
          5         line = thefile.readline()
          6         if not line:
    ----> 7             time.sleep(0.1)
          8             continue
          9         yield line


    KeyboardInterrupt: 



```python
def grep(pattern, lines):
    for line in lines:
        if pattern in line:
            yield line
# tail -f | grep python
logfile = open("access-log")
loglines = follow(logfile)
pylines = grep("python", loglines)

for line in pylines:
    print line
```

    python
    



    ---------------------------------------------------------------------------

    KeyboardInterrupt                         Traceback (most recent call last)

    <ipython-input-12-cfb4e4626dc8> in <module>()
          8 pylines = grep("python", loglines)
          9 
    ---> 10 for line in pylines:
         11     print line


    <ipython-input-12-cfb4e4626dc8> in grep(pattern, lines)
          1 def grep(pattern, lines):
    ----> 2     for line in lines:
          3         if pattern in line:
          4             yield line
          5 # tail -f | grep python


    <ipython-input-11-7b507f742f61> in follow(thefile)
          5         line = thefile.readline()
          6         if not line:
    ----> 7             time.sleep(0.1)
          8             continue
          9         yield line


    KeyboardInterrupt: 


# 协程(coroutines)


```python
def consumer():
    r = 'initial'
    while True:
        n = yield r
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'

def produce(c):
    '''
    为什么启动时候需要send(None)?
    新建一个生成器之后，函数并没有执行，此时，生成器并没有处于yield状态即断点状态，不需要接受传递值，因此传入None，并使其执行到yield进入断点状态。
    本例中c.send(None)会使得函数执行到 "n = yield r", 也就是抛出一个 r 给发送者作为返回值，然后中断。当再一次调用c.send(1)的时候，生成器从断点恢复，，
    首先接受发送过来的值1，执行 n = 1 的赋值语句，然后继续往下执行，直到循环回来再一次碰到yield，抛出r='200 OK'.然后中断进入断点状态(这个状态就是等待一个接受值，然后从断点语句 n = receive开始往后执行)。
    '''
    print c.send(None)    # 生成器刚刚启动的时候，只能传递None，否则TypeError: can't send non-None value to a just-started generator
    # print next(c)
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()

c = consumer()
produce(c)
```

    initial
    [PRODUCER] Producing 1...
    [CONSUMER] Consuming 1...
    [PRODUCER] Consumer return: 200 OK
    [PRODUCER] Producing 2...
    [CONSUMER] Consuming 2...
    [PRODUCER] Consumer return: 200 OK
    [PRODUCER] Producing 3...
    [CONSUMER] Consuming 3...
    [PRODUCER] Consumer return: 200 OK
    [PRODUCER] Producing 4...
    [CONSUMER] Consuming 4...
    [PRODUCER] Consumer return: 200 OK
    [PRODUCER] Producing 5...
    [CONSUMER] Consuming 5...
    [PRODUCER] Consumer return: 200 OK


>为什么启动时候需要send(None)?
    
    新建一个生成器之后，函数并没有执行，此时，生成器并没有处于yield状态即断点状态，不需要接受传递值，因此传入None，并使其执行到yield进入断点状态。
    
    本例中c.send(None)会使得函数执行到 "n = yield r", 也就是抛出一个 r 给发送者作为返回值，然后中断。当再一次调用c.send(1)的时候，生成器从断点恢复，，
    
    首先接受发送过来的值1，执行 n = 1 的赋值语句，然后继续往下执行，直到循环回来再一次碰到yield，抛出r='200 OK'.然后中断进入断点状态(这个状态就是等待一个接受值，然后从断点语句 n = receive开始往后执行)。
    
    由于next也能让生成器开始执行，因此启动生成器的时候，使用next() 和 send(None) 是一样的效果。

有经常需要先 generator.next() 或者 generator.send(None) 启动一个协程，很容易忘记，因此可以自己写一个装饰器来用。


```python
def coroutine(func):
    def start(*args, **kwargs):
        cr = func(*args, **kwargs)
        cr.next()
        return cr
    return start

@coroutine
def grep(pattern):
    print "Looking for %s" % pattern
    try:
        while True:
            line = (yield)
            if pattern in line:
                print line,
    except GeneratorExit:
        print "Going away. Goodbye"
```


```python
g = grep("python")
g.send("python generator rock!")
```

    Looking for python
    python generator rock!


抛出异常，可以使用throw() 函数。


```python
g.throw(RuntimeError, "You're hosed")
```


    ---------------------------------------------------------------------------

    RuntimeError                              Traceback (most recent call last)

    <ipython-input-17-83761f9b0edc> in <module>()
    ----> 1 g.throw(RuntimeError, "You're hosed")
    

    <ipython-input-14-34c09fc3efc5> in grep(pattern)
         11     try:
         12         while True:
    ---> 13             line = (yield)
         14             if pattern in line:
         15                 print line,


    RuntimeError: You're hosed



```python
def down(n):
    print "countingd down from", n
    while n >= 0:
        newvalue = (yield n)
        if newvalue is not None:
            n = newvalue
        else:
            n -= 1
c = down(5)
for n in c:
    print n
    if n == 5:
        c.send(3)
```

    countingd down from 5
    5
    2
    1
    0


 - Generators produce data for iteration
 - Coroutines are consumers of data
 


```python
import time
def follow(thefile, target):
    thefile.seek(0,2)
    while True:
        line = thefile.readline()
        if not line:
            time.sleep(0.1)
            continue
        target.send(line)

@coroutine
def printer():
    while True:
        line = (yield)
        print line,

f = open("access-log")
follow(f, printer())
f.close()
```

    python



    ---------------------------------------------------------------------------

    KeyboardInterrupt                         Traceback (most recent call last)

    <ipython-input-20-27b5a2021e19> in <module>()
         16 
         17 f = open("access-log")
    ---> 18 follow(f, printer())
         19 f.close()


    <ipython-input-20-27b5a2021e19> in follow(thefile, target)
          5         line = thefile.readline()
          6         if not line:
    ----> 7             time.sleep(0.1)
          8             continue
          9         target.send(line)


    KeyboardInterrupt: 



```python
@coroutine
def grep(pattern, target):
    while True:
        line = (yield)
        if pattern in line:
            target.send(line)

f = open("access-log")
follow(f, grep('python', printer()))
f.close()
```


```python
@coroutine
def broadcast(targets):
    while True:
        item = (yield)
        for target in targets:
            target.send(item)

f = open("access-log")
follow(f, 
       broadcast([
           grep('python', printer()),
           grep('rnn', printer()),
           grep('cnn', printer())
       ])
)
```

# 协程实现多任务


```python
class Task(object):
    taskid = 0
    
    def __init__(self, target):
        Task.taskid += 1
        self.tid = Task.taskid
        self.target = target
        self.sendval = None
    
    def run(self):
        return self.target.send(self.sendval)
```


```python
from Queue import Queue
class Scheduler(object):
    def __init__(self):
        self.ready = Queue() # 所有等待调度的Task
        self.taskmap = {} # 跟踪所有活跃状态的Task
    
    def new(self, target):
        newtask = Task(target) # create a new task
        self.taskmap[newtask.tid] = newtask # keep track
        self.schedule(newtask) # 调度，即放入ready队列中
        return newtask.tid
    
    def schedule(self, task):
        self.ready.put(task)
    
    def mainloop(self):
        while self.taskmap:
            task = self.ready.get() # 从队列中取出任务调度
            result = task.run() # 执行任务到 yield
            self.schedule(task) # 调度任务
```


```python
def foo():
    while True:
        print "foo"
        yield
        
def bar():
    while True:
        print "bar"
        yield

sched = Scheduler()
sched.new(foo())
sched.new(bar())
#sched.mainloop()
```




    2



以上的调度器有两个问题，第一，任务无法终止；第二，某个任务如果返回，那么调度器会崩溃。


```python
def foo1():
    for i in range(3):
        print "foo1"
        yield
sched = Scheduler()
sched.new(foo1())
sched.new(bar())
sched.mainloop()
```

    foo1
    bar
    foo1
    bar
    foo1
    bar



    ---------------------------------------------------------------------------

    StopIteration                             Traceback (most recent call last)

    <ipython-input-37-d6d5e458832d> in <module>()
          6 sched.new(foo1())
          7 sched.new(bar())
    ----> 8 sched.mainloop()
    

    <ipython-input-36-f7715003ba50> in mainloop(self)
         17         while self.taskmap:
         18             task = self.ready.get() # 从队列中取出任务调度
    ---> 19             result = task.run() # 执行任务到 yield
         20             self.schedule(task) # 调度任务


    <ipython-input-32-ee4ede936ba7> in run(self)
          9 
         10     def run(self):
    ---> 11         return self.target.send(self.sendval)
    

    StopIteration: 



```python
from Queue import Queue
class Scheduler(object):
    def __init__(self):
        self.ready = Queue() # 所有等待调度的Task
        self.taskmap = {} # 跟踪所有活跃状态的Task
    
    def new(self, target):
        newtask = Task(target) # create a new task
        self.taskmap[newtask.tid] = newtask # keep track
        self.schedule(newtask) # 调度，即放入ready队列中
        return newtask.tid
    
    def schedule(self, task):
        self.ready.put(task)
    
    def exit(self, task):
        print "Task %d terminated" % task.tid
        del self.taskmap[task.tid]
    
    def mainloop(self):
        while self.taskmap:
            task = self.ready.get() # 从队列中取出任务调度
            try:
                result = task.run() # 执行任务到 yield
            except StopIteration:
                self.exit(task)
                continue
            self.schedule(task) # 调度任务
```


```python
def ping():
    for i in range(4):
        print "ping"
        yield

def pong():
    for i in range(8):
        print "pong"
        yield
sched = Scheduler()
sched.new(ping())
sched.new(pong())
sched.mainloop()
```

    ping
    pong
    ping
    pong
    ping
    pong
    ping
    pong
    Task 7 terminated
    pong
    pong
    pong
    pong
    Task 8 terminated



```python
class SystemCall(object):
    def handle(self):
        pass

class GetTid(SystemCall):
    def handle(self):
        self.task.sendval = self.task.tid
        self.sched.schedule(self.task)
        
from Queue import Queue
class Scheduler(object):
    def __init__(self):
        self.ready = Queue() # 所有等待调度的Task
        self.taskmap = {} # 跟踪所有活跃状态的Task
    
    def new(self, target):
        newtask = Task(target) # create a new task
        self.taskmap[newtask.tid] = newtask # keep track
        self.schedule(newtask) # 调度，即放入ready队列中
        return newtask.tid
    
    def schedule(self, task):
        self.ready.put(task)
    
    def exit(self, task):
        print "Task %d terminated" % task.tid
        del self.taskmap[task.tid]
    
    def mainloop(self):
        while self.taskmap:
            task = self.ready.get() # 从队列中取出任务调度
            try:
                result = task.run() # 执行任务到 yield
                if isinstance(result, SystemCall):
                    result.task = task
                    result.sched = self
                    result.handle()
                    continue
            except StopIteration:
                self.exit(task)
                continue
            self.schedule(task) # 调度任务
            
def didi():
    mytid = yield GetTid()
    for i in range(5):
        print "didi", mytid
        yield

def dada():
    mytid = yield GetTid()
    for i in range(10):
        print "dada", mytid
        yield
sched = Scheduler()
sched.new(didi())
sched.new(dada())
sched.mainloop()
```

    didi 9
    dada 10
    didi 9
    dada 10
    didi 9
    dada 10
    didi 9
    dada 10
    didi 9
    dada 10
    Task 9 terminated
    dada 10
    dada 10
    dada 10
    dada 10
    dada 10
    Task 10 terminated



```python
# ------------------------------------------------------------
# pyos5.py  -  The Python Operating System
#
# Step 5: Added system calls for simple task management
# ------------------------------------------------------------

# ------------------------------------------------------------
#                       === Tasks ===
# ------------------------------------------------------------
class Task(object):
    taskid = 0
    def __init__(self,target):
        Task.taskid += 1
        self.tid     = Task.taskid   # Task ID
        self.target  = target        # Target coroutine
        self.sendval = None          # Value to send

    # Run a task until it hits the next yield statement
    def run(self):
        return self.target.send(self.sendval)

# ------------------------------------------------------------
#                      === Scheduler ===
# ------------------------------------------------------------
from Queue import Queue

class Scheduler(object):
    def __init__(self):
        self.ready   = Queue()   
        self.taskmap = {}        

    def new(self,target):
        newtask = Task(target)
        self.taskmap[newtask.tid] = newtask
        self.schedule(newtask)
        return newtask.tid

    def exit(self,task):
        print "Task %d terminated" % task.tid
        del self.taskmap[task.tid]

    def schedule(self,task):
        self.ready.put(task)

    def mainloop(self):
         while self.taskmap:
            task = self.ready.get()
            try:
                result = task.run()
                if isinstance(result,SystemCall):
                    result.task  = task
                    result.sched = self
                    result.handle()
                    continue
            except StopIteration:
                self.exit(task)
        # ------------------------------------------------------------
# pyos5.py  -  The Python Operating System
#
# Step 5: Added system calls for simple task management
# ------------------------------------------------------------

# ------------------------------------------------------------
#                       === Tasks ===
# ------------------------------------------------------------
class Task(object):
    taskid = 0
    def __init__(self,target):
        Task.taskid += 1
        self.tid     = Task.taskid   # Task ID
        self.target  = target        # Target coroutine
        self.sendval = None          # Value to send

    # Run a task until it hits the next yield statement
    def run(self):
        return self.target.send(self.sendval)

# ------------------------------------------------------------
#                      === Scheduler ===
# ------------------------------------------------------------
from Queue import Queue

class Scheduler(object):
    def __init__(self):
        self.ready   = Queue()   
        self.taskmap = {}        

    def new(self,target):
        newtask = Task(target)
        self.taskmap[newtask.tid] = newtask
        self.schedule(newtask)
        return newtask.tid

    def exit(self,task):
        print "Task %d terminated" % task.tid
        del self.taskmap[task.tid]

    def schedule(self,task):
        self.ready.put(task)

    def mainloop(self):
         while self.taskmap:
            task = self.ready.get()
            try:
                result = task.run()
                if isinstance(result,SystemCall):
                    result.task  = task
                    result.sched = self
                    result.handle()
                    continue
            except StopIteration:
                self.exit(task)
                continue
            self.schedule(task)

# ------------------------------------------------------------
#                   === System Calls ===
# ------------------------------------------------------------

class SystemCall(object):
    def handle(self):
        pass

# Return a task's ID number
class GetTid(SystemCall):
    def handle(self):
        self.task.sendval = self.task.tid
        self.sched.schedule(self.task)

# Create a new task
class NewTask(SystemCall):
    def __init__(self,target):
        self.target = target
    def handle(self):
        tid = self.sched.new(self.target)
        self.task.sendval = tid
        self.sched.schedule(self.task)

# Kill a task
class KillTask(SystemCall):
    def __init__(self,tid):
        self.tid = tid
    def handle(self):
        task = self.sched.taskmap.get(self.tid,None)
        if task:
            task.target.close() 
            self.task.sendval = True
        else:
            self.task.sendval = False
        self.sched.schedule(self.task)


# ------------------------------------------------------------
#                   === System Calls ===
# ------------------------------------------------------------

class SystemCall(object):
    def handle(self):
        pass

# Return a task's ID number
class GetTid(SystemCall):
    def handle(self):
        self.task.sendval = self.task.tid
        self.sched.schedule(self.task)

# Create a new task
class NewTask(SystemCall):
    def __init__(self,target):
        self.target = target
    def handle(self):
        tid = self.sched.new(self.target)
        self.task.sendval = tid
        self.sched.schedule(self.task)

# Kill a task
class KillTask(SystemCall):
    def __init__(self,tid):
        self.tid = tid
    def handle(self):
        task = self.sched.taskmap.get(self.tid,None)
        if task:
            task.target.close() 
            self.task.sendval = True
        else:
            self.task.sendval = False
        self.sched.schedule(self.task)

# ------------------------------------------------------------
#                      === Example ===
# ------------------------------------------------------------
if __name__ == '__main__':
    def foo():
        mytid = yield GetTid()
        while True:
            print "I'm foo", mytid
            yield

    def main():
        child = yield NewTask(foo())    # Launch new task
        print 'child', child
        for i in xrange(5):
            print "i'm main", i
            yield
        yield KillTask(child)           # Kill the task
        print "main done"

    sched = Scheduler()
    sched.new(main())
    sched.mainloop()
```

    child 2
    i'm main 0
    I'm foo 2
    i'm main 1
    I'm foo 2
    i'm main 2
    I'm foo 2
    i'm main 3
    I'm foo 2
    i'm main 4
    I'm foo 2
    Task 2 terminated
    main done
    Task 1 terminated


添加一个等待系统调用，此时需要一个记录{waited_task_id,[waiting_task_id]}的词典记录所有的等待被等待关系，并且使用一个 waiting 队列调度所有的等待任务。被等待任务在退出的时候，必须通知等待的任务即重新调度执行等待的任务。


```python
# ------------------------------------------------------------
# pyos6.py  -  The Python Operating System
#
# Added support for task waiting
# ------------------------------------------------------------

# ------------------------------------------------------------
#                       === Tasks ===
# ------------------------------------------------------------
class Task(object):
    taskid = 0
    def __init__(self,target):
        Task.taskid += 1
        self.tid     = Task.taskid   # Task ID
        self.target  = target        # Target coroutine
        self.sendval = None          # Value to send

    # Run a task until it hits the next yield statement
    def run(self):
        return self.target.send(self.sendval)

# ------------------------------------------------------------
#                      === Scheduler ===
# ------------------------------------------------------------
from Queue import Queue

class Scheduler(object):
    def __init__(self):
        self.ready   = Queue()   
        self.taskmap = {}        

        # Tasks waiting for other tasks to exit
        self.exit_waiting = {}

    def new(self,target):
        newtask = Task(target)
        self.taskmap[newtask.tid] = newtask
        self.schedule(newtask)
        return newtask.tid

    def exit(self,task):
        print "Task %d terminated" % task.tid
        del self.taskmap[task.tid]
        # Notify other tasks waiting for exit
        for task in self.exit_waiting.pop(task.tid,[]):
            self.schedule(task)

    def waitforexit(self,task,waittid):
        if waittid in self.taskmap:
            self.exit_waiting.setdefault(waittid,[]).append(task)
            return True
        else:
            return False

    def schedule(self,task):
        self.ready.put(task)

    def mainloop(self):
         while self.taskmap:
            task = self.ready.get()
            try:
                result = task.run()
                if isinstance(result,SystemCall):
                    result.task  = task
                    result.sched = self
                    result.handle()
                    continue
            except StopIteration:
                self.exit(task)
                continue
            self.schedule(task)

# ------------------------------------------------------------
#                   === System Calls ===
# ------------------------------------------------------------

class SystemCall(object):
    def handle(self):
        pass

# Return a task's ID number
class GetTid(SystemCall):
    def handle(self):
        self.task.sendval = self.task.tid
        self.sched.schedule(self.task)

# Create a new task
class NewTask(SystemCall):
    def __init__(self,target):
        self.target = target
    def handle(self):
        tid = self.sched.new(self.target)
        self.task.sendval = tid
        self.sched.schedule(self.task)

# Kill a task
class KillTask(SystemCall):
    def __init__(self,tid):
        self.tid = tid
    def handle(self):
        task = self.sched.taskmap.get(self.tid,None)
        if task:
            task.target.close() 
            self.task.sendval = True
        else:
            self.task.sendval = False
        self.sched.schedule(self.task)

# Wait for a task to exit
class WaitTask(SystemCall):
    def __init__(self,tid):
        self.tid = tid
    def handle(self):
        result = self.sched.waitforexit(self.task,self.tid)
        self.task.sendval = result
        # If waiting for a non-existent task,
        # return immediately without waiting
        if not result:
            self.sched.schedule(self.task)

# ------------------------------------------------------------
#                      === Example ===
# ------------------------------------------------------------
if __name__ == '__main__':
    def foo():
        for i in xrange(5):
            print "I'm foo"
            yield

    def main():
        child = yield NewTask(foo())
        print "Waiting for child"
        yield WaitTask(child)
        print "Child done"

    sched = Scheduler()
    sched.new(main())
    sched.mainloop()
```

    I'm foo
    Waiting for child
    I'm foo
    I'm foo
    I'm foo
    I'm foo
    Task 2 terminated
    Child done
    Task 1 terminated


>One of the most critical parts of writing an OS is both protecting the system and keeping user applications isolated. In the sample code, the Scheduler class represents a kind of "operating system." You will notice that the tasks defined in this course never directly interact with this scheduler (other than using yield to execute scheduler traps). That is, they do not hold references to the scheduler object, they do not invoke methods on the scheduler, they do not inspect internal scheduler data, and they don't hold references to other tasks. For all practical purposes, the scheduler and the tasks are two completely different execution domains. There is a good reason for keeping this separation. Namely it promotes a loose coupling between tasks and their execution environment. One could imagine creating other kinds of task schedulers that run tasks within threads or subprocesses. If you've set things up right and taken great care to promote the separation of tasks and scheduling, such schedulers would be able to run existing tasks without modification.

操作系统和应用的分离，在任务代码中，没有任何与scheduler直接交互的地方，即没有直接调用内核的代码，而是通过内核提供的接口SystemCall完成。或者通过简单的 yield 内陷trap。


```python

def main():
    r = yield mul_add(2,2,3)
    print r
    yield

def mul_add(x,y,z):
    r = yield add(x,y)
    yield r*z

def add(x,y):
    yield x+y
    
m = main()
sub = m.send(None)
subsub = sub.send(None)
rsubsub = subsub.send(None)
rsub = sub.send(rsubsub)
r = m.send(rsub)
r
```

    12

