---
title: "python-thread-locals"
date: 2018-04-28 18:32
---

[TOC]

# 什么是线程局部存储(ThreadLocal)
这个概念最早是相对于全局变量来说的，就是我们在编程的时候，会涉及到希望所有线程都能够共享访问同一个变量，在 Python/Go/C 中，我们就可以定义一个全局变量，这样Global Variable 对多个线程就是可见的，大家都可以操作。例如，一个全局的配置变量或单实例对象，所有线程就可以很方便访问了，但是仅仅这样有一个前提，就是这个变量的并发操作必须是幂等的，读写不影响我们程序的正确性。但是往往多线程共同操作一个全局变量，就会影响程序的正确性，因此我们必须枷锁，比如经典的并发加操作。
```python
import threading
count = 0
lock = threading.RLock()
def inc():
    global count, lock
    with lock:
        count += 1
```
上面那个例子很多博客用来做ThreadLocal变量的讲解，实际上我觉得是有误导的，不恰当的。因为这种共享变量，你必须枷锁，因为他的目的就是为了大家一起去更新一个共享变量，多线程环境下必须枷锁。就算你使用ThreadLocal替换也没用，ThreadLocal能替换这个Count变量让所有线程单独存储一份么，不满足需求。你单独存一份，更改之后还得把结果再次写回到全局变量去更新，那写会的过程还是得枷锁。除非使用Golang中的单Channel更新机制，才能避免枷锁。

所以ThreadLocal变量使用强调的侧重点不在这里，更多的是在编程范式上面。其实就是有些时候，我们某个变量类型很多函数或者类都需要用，但是我又不想写死在代码里，每次传递参数都要传递这个类或者变量，因为一旦这个类发生类型上的变化，可能对于静态类型的语言，很多地方就得修改参数，而且这种变量一直在程序代码的参数传递中层层出现，你如果写过代码就会有感觉，有时候你设计的函数API好像一层层的得把一个参数传递进去，即使某些层好像用不到这个参数。
```python
def getMysqlConn(passwd, db, host="localhost", port=3306, user="root", charset='utf8'):
    conn = MySQLdb.connect(host=host, port=port, user=user, passwd=passwd, db=db, charset=charset)
    return conn

def func1(zzz, passwd, db, host="localhost", port=3306, user="root", charset='utf8'):
    conn = getMysqlConn(passwd, db, host, port, user, charset)
    ...

def func2(xxx,yyy,zzz, passwd, db, host="localhost", port=3306, user="root", charset='utf8'):
    ...
    func1(zzz,passwd,db,host,port,user,charset)
```

上面的代码你可能会疯掉。那么你可能就考虑想把这个参数提出来，当成全局变量算了，哪一层用到了直接用就好了，不能让他无缘无故的不停的被当成局部变量传参。文章[Alternatives to global variables and passing the same value over a long chain of calls](https://tomassetti.me/alternatives-to-global-variables-and-passing-the-same-value-over-a-long-chain-of-calls/)描述了这个问题，但是这个时候出现的问题就是，可能其他代码线程会不可控的更改这个变量，导致你的程序发生未知错误。你把这种参数变成全局的暴露出来，那么基于的假设就是该参数不会被随意修改！一旦这个假设崩塌，你的程序可能会发生灾难后果。这不符合软件设计的开闭原则。所以我们使用TLS技术化解这种矛盾。

那么我们就设计了一种方案，就是有这样一种变量，他是全局的，但是每个线程在访问的时候都会存储一份成为自己的局部变量，修改就不会相互影响了。比如 Linux/Unix 的 C 程序库 libc 的全局变量`errno`, 这个其实就是TLS的例子。当系统调用从内核空间返回用户空间时，如果系统调用出错，那么便设置`errno`的值为一个负值，这样就不需要每次在函数内部定义局部变量。但是当多线程的概念和技术被提出后，这套机制就不再适用了，可以使用局部变量，但是不太可能去更改已有的代码了，比较好的解决方案是让每个线程都有自己的`errno`。实际上，现在的C库函数不是把出错代码写入全局量`errno`，而是通过一个函数`__errno_location()`获取一个地址，再把出错代码写入该地址，其意图就是让不同的线程使用不同的出错代码存储地点，而`errno`，现在一般已经变成了一个宏定义。每一个线程都会维护自己的一份，修改不影响其他线程。


这是不是意味着ThreadLocal对象不用枷锁了？ 其实这个ThreadLocal和同步没有关系，他仅仅是提供了一种方便每个线程快速访问变量的方式，但是如果这个对象本身有些共享状态需要大家一起维护(比如Count++)，你就必须枷锁，尽管每个线程操作的是ThreadLocal副本。[维基百科](https://en.wikipedia.org/wiki/Thread-local_storage)上有以下原话：

> A second use case would be multiple threads accumulating information into a global variable. To avoid a race condition, every access to this global variable would have to be protected by a mutex. Alternatively, each thread might accumulate into a thread-local variable (that, by definition, cannot be read from or written to from other threads, implying that there can be no race conditions). **Threads then only have to synchronise a final accumulation from their own thread-local variable into a single, truly global variable.**

比如我们写了一个共享的Manager类，这个类可能是用来做数据库连接，网络连接或者其他的做底层管理功能。我们有很多线程需要使用这个Manager的某些功能，并且这种类不是用来表示一种状态，供所有线程并发修改其状态并将最终修改的结果表现在该类上面（上面count的例子）。Manager只是可以提供给线程使用某些功能，然后每个线程可以把这个Manager复制一份成为自己的局部变量，自己可以随意修改，但是不会影响到其他线程，因为是复制的一份。但是如果你需要让管理器记录所有的连接操作次数，那么多线程对立面的某些变量访问比如Count就需要枷锁了。

# TLS 在Python中的运用和实现
## 简单实用
 ThreadLocal不仅仅可以解决全局变量访问冲突，其实还有其他好处，在[PEP266](https://www.python.org/dev/peps/pep-0266/)中有提到，ThreadLocal变量是可以减少指令加速运算的，因为全局变量往往需要更多的指令(需要for loop)来做查询访问，而ThreadLocal 之后，有了索引表，直接可以一条指令找到这个对象。

```python
import threading

userName = threading.local()

def SessionThread(userName_in):
    userName.val = userName_in
    print(userName.val)   

Session1 = threading.Thread(target=SessionThread("User1"))
Session2 = threading.Thread(target=SessionThread("User2"))

# start the session threads
Session1.start()
Session2.start()
# wait till the session threads are complete
Session1.join()
Session2.join()
```
上述Threadlocal的实现原理类似有一个全局的词典，词典的key是线程id，value就是共享的全局变量的副本。每次访问全局变量的时候，你访问到的其实是副本，只是Python使用黑魔法帮我们屏蔽了这个 `userName.val` 的访问细节，其实他访问的是词典中的对应线程所拥有的对象副本。

## 实现源码分析
```python
__all__ = ["local"]
class _localbase(object):
    __slots__ = '_local__key', '_local__args', '_local__lock'

    def __new__(cls, *args, **kw):
        # 新建一个类对象
        self = object.__new__(cls)
        # 在主线程中初始化这个这个全局对象的某些属性，比如 `_local__key`, 这个key以后会用作其他线程使用全局变量副本的查询依据，以后每个线程都会根据这个key来查找自己的局部副本数据
        key = '_local__key', 'thread.local.' + str(id(self))
        object.__setattr__(self, '_local__key', key)
        object.__setattr__(self, '_local__args', (args, kw))
        # 多线程会并发设置全局变量的属性，这时候会并发访问设置属性，因此需要一把全局锁，进行互斥操作
        object.__setattr__(self, '_local__lock', RLock())

        if (args or kw) and (cls.__init__ is object.__init__):
            raise TypeError("Initialization arguments are not supported")

        # We need to create the thread dict in anticipation of
        # __init__ being called, to make sure we don't call it
        # again ourselves.
        dict = object.__getattribute__(self, '__dict__')
        current_thread().__dict__[key] = dict

        return self

def _patch(self):
    # 拿到全局的key
    key = object.__getattribute__(self, '_local__key')
    # 在当前线程中根据key找到线程的私有数据副本，并替换掉 ThreadLocal自己的__dict__属性。如果没有，就创建一个，并添加
    d = current_thread().__dict__.get(key)
    if d is None:
        d = {}
        # 线程还没得私有数据副本，创建一个并加入线程自己的属性中
        current_thread().__dict__[key] = d
        # 替换ThreadLocal的__dict__为当前线程的私有数据词典d
        object.__setattr__(self, '__dict__', d)

        # we have a new instance dict, so call out __init__ if we have
        # one
        # 这段的意思其实是，如果原来的全局变量ThreadLocal 本身有一些其他的属性和数据，那么直接替换掉一个新dict之后，以前的数据就丢失了，这里我们必须初始化以前的数据到新dict中
        cls = type(self)
        if cls.__init__ is not object.__init__:
            args, kw = object.__getattribute__(self, '_local__args')
            cls.__init__(self, *args, **kw)
    else:
        object.__setattr__(self, '__dict__', d)

class local(_localbase):

    def __getattribute__(self, name):
        lock = object.__getattribute__(self, '_local__lock')
        lock.acquire()
        try:
            _patch(self)
            return object.__getattribute__(self, name)
        finally:
            lock.release()

    def __setattr__(self, name, value):
        if name == '__dict__':
            raise AttributeError(
                "%r object attribute '__dict__' is read-only"
                % self.__class__.__name__)
        # 拿到早已经在主线程设置的共享的一把锁
        lock = object.__getattribute__(self, '_local__lock')
        lock.acquire()
        try:
            _patch(self)# 关键代码，这个patch会导致 Threadlocal 这个数据的__dict__直接被换成了所在线程自己的私有数据, Python 里面有很多这种patch的替换手段，就是直接把基础库的某些功能和函数直接替换成了第三方库的比如monkey patch
            # 再次设置属性的时候，设置的__dict__ 其实不是 Threadlocal 自己的属性了，是而是当前所在线程的__dict__的某一个key-value 副本数据的value，这个value是一个dict
            # object 的setattr默认行为其实就是在自己的__dict__对象中添加一对key-pair，但是现在他的__dict__已经更换成所在线程的一个数据副本词典了，黑魔法替换就在这里
            return object.__setattr__(self, name, value)
        finally:
            lock.release()

    def __delattr__(self, name):
        if name == '__dict__':
            raise AttributeError(
                "%r object attribute '__dict__' is read-only"
                % self.__class__.__name__)
        lock = object.__getattribute__(self, '_local__lock')
        lock.acquire()
        try:
            _patch(self)
            return object.__delattr__(self, name)
        finally:
            lock.release()

    def __del__(self):
        import threading

        key = object.__getattribute__(self, '_local__key')

        try:
            # We use the non-locking API since we might already hold the lock
            # (__del__ can be called at any point by the cyclic GC).
            threads = threading._enumerate()
        except:
            # If enumerating the current threads fails, as it seems to do
            # during shutdown, we'll skip cleanup under the assumption
            # that there is nothing to clean up.
            return

        for thread in threads:
            try:
                __dict__ = thread.__dict__
            except AttributeError:
                # Thread is dying, rest in peace.
                continue

            if key in __dict__:
                try:
                    del __dict__[key]
                except KeyError:
                    pass # didn't have anything in this thread

from threading import current_thread, RLock

data = local()
print data.__dict__
def t(x):
    global data
    data.x = x
    data.y = 1
    print current_thread().__dict__
    print data.__dict__
    print

    
t1 = threading.Thread(target=t, args = (777,))
t2 = threading.Thread(target=t, args = (888,))
print current_thread().__dict__
t1.start()
t2.start()
t1.join()
t2.join()
print data.__dict__
```
关键技术就在patch上面，据, Python 里面有很多这种patch的替换手段，就是直接把基础库的某些功能和函数直接替换成了第三方库的比如monkey patch. 再次设置属性的时候，设置的 `__dict__` 其实不是 `ThreadLocal` 自己的，是而是当前所在线程的 `__dict__` 的某一个 `key-value` 副本数据， `key` 就是线程访问的某个TLS变量生成的（一个线程可以有很多TLS变量，每个有不同的key），`value`是一个`dict`. `object` 的 `setattr` 默认行为其实就是在自己的`__dict__` 对象中添加一对key-pair，但是现在他的`__dict__`已经更换成所在线程的一个数据副本词典`dict`了，黑魔法替换就在这里.

下面的例子展示了Python黑魔法的一个替换词典的方式，可以运行看看
```python
class A(object):
    def substitute(self, d):
        object.__setattr__(self, '__dict__', d)
a = A()
a.y  = 3
old_dict = a.__dict__
print old_dict
d = {'x':1}
a.substitute(d)
print a.__dict__
a.y = 777
print a.__dict__
print d
```
如果A本身已经含有一些数据，那就不能简单的直接复制了，还需要初始化以前的数据填充新的词典，这也是在源码中看到的。
```python
from threading import current_thread
class A(object):
    def __new__(cls, *args, **kw):
        self = object.__new__(cls)
        setattr(cls, '_local__args', (args, kw))
        return self
        
    def __init__(self, *args, **kw):
        self.shared_x = kw["shared_x"]
        self.shared_y = kw["shared_y"]
    def substitute(self, d):
        object.__setattr__(self, '__dict__', d)
        cls = type(self)
        if cls.__init__ is not object.__init__:
            print "7---------------"
            args, kw = getattr(self, '_local__args')
            cls.__init__(self, *args, **kw)
        
a = A(shared_x=111, shared_y=222)
a.y  = 3
old_dict = a.__dict__
print old_dict
d = {'x':1}
a.substitute(d)
print a.__dict__
a.y = 777
print a.__dict__
print d
print old_dict
```

下图就是访问每个线程访问过程，实际上操作的是线程自己的私有数据副本。同时需要注意的还是那句话，**使用 ThreadLocal 对象不意味着你的程序不需要再枷锁**，比如这个 ThreadLocal 对象可能又引用了其他共享状态的对象，那么就要对这个共享状态对象的操作进行枷锁实现同步和互斥。
<img src="/static/images/Python/Core/threadlocal.png" style="width:700px;height:300px;">
<caption><center> ThreadLocal 实现过程 </center></caption>


# TLS 在Java 中的运用和实现
## 简单实用
```java
public class ThreadLocalExample {

    public static class MyRunnable implements Runnable {

        private ThreadLocal threadLocal = new ThreadLocal();

        @Override
        public void run() {
            threadLocal.set((int) (Math.random() * 100D));
            try {
            Thread.sleep(2000);
            } catch (InterruptedException e) {

            }
            System.out.println(threadLocal.get());
        }
    }

    public static void main(String[] args) {
         MyRunnable sharedRunnableInstance = new MyRunnable();
         Thread thread1 = new Thread(sharedRunnableInstance);
         Thread thread2 = new Thread(sharedRunnableInstance);
         thread1.start();
         thread2.start();
    }
}
```

## 源码实现
有了Python版本的分析，Java版本就不再多做解释，感兴趣的可以看看源码，实现原理肯定都是大同小异，只是语言上的差异，导致 Java 不可能像Python这种动态类型语言一样灵活。 

**需要每个线程都维护一个 key-value 集合数据结构，记录每个线程访问到的 TLS 变量副本，这样每个线程可以根据 key 来找到相应的 TLS副本数据，对副本数据进行真实的操作，而不是TLS全局变量或者静态类（Java中）**。在Python中直接很简单的使用了动态数据绑定的词典数据结构，在Java中稍显麻烦，需要实现一个类似Map的结构，ThreadLocal.get() 方法其实本质上也是和Python中一样，先获取当前线程自己的ThreadLocalMap对象（就是每个线程维护的TLS key-value集合啦）。再从ThreadLocalMap对象中找出当前的ThreadLocal变量副本，和HashMap一样的采用了链地址法的hash结构。可以参考文章[Java 多线程（7）： ThreadLocal 的应用及原理](https://segmentfault.com/a/1190000010251063)。Java 里一般是采用泛型规定你共享的变量类型，然后每个线程维护该变量的副本。


# 小结
TLS技术的使用和属性：
 - 解决多线程编程中的**对同一变量的访问冲突**的一种技术，TLS会为**每一个线程维护一个和该线程绑定的变量的副本**。而**不是无止尽的传递局部参数的方式编程**。
 - 每一个线程都拥有自己的变量副本，**并不意味着就一定不会对TLS变量中某些操作枷锁了**。
 - Java平台的`java.lang.ThreadLocal`是TLS技术的一种实现， Python 中有`thread.local()`。
 - TLS使用的缺陷是，如果你的线程都不退出，那么**副本数据可能一直不被GC回收，会消耗很多资源**，比如线程池中，线程都不退出，使用TLS需要非常小心。

TLS技术的实现原理：
**需要每个线程都维护一个 key-value 集合数据结构，记录每个线程访问到的 TLS 变量副本，这样每个线程可以根据 key 来找到相应的 TLS副本数据，对副本数据进行真实的操作，而不是TLS全局变量或者静态类（Java中）**。

**TLS 变量自己会根据当前调用他的Thread对象，根据Thread对象得到该线程维护的 TLS 副本集合，然后进一步根据当前TLS的key，查到到key对一个的TLS副本数据。这样就给每个线程造成一种假象，以为大家可以同时更新一个全局共享变量或者静态类对象。**

# 参考
 - [Java 多线程（7）： ThreadLocal 的应用及原理](https://segmentfault.com/a/1190000010251063)