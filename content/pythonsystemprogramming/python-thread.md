---
title: "python-thread"
date: 2017-12-05 22:22
---
# python thread
python 中有 thread 和 threading两个模块， thread是比较基础的模块，可以参考[文档](https://docs.python.org/2/library/thread.html#module-thread)只提供了几个最基础的线程功能，分配锁和开启新线程； threading 模块就是在 thread 基础上封装出来的更友好的线程模块。 下面是 thread 的例子，
```python
import thread
import time

a_lock = thread.allocate_lock()

with a_lock:
    print "a_lock is locked while this executes"
    
def do_something_in_new_thread(tname, number):
    print "Hello, I am in thread", tname, number
    print "I will interrupt main thread after 5 sec."
    time.sleep(5)
    thread.interrupt_main()

thread.start_new_thread(do_something_in_new_thread, ("thread-1", 1))

while True:
    print "I am main thread, I will sleep 2 sec."
    time.sleep(2)

```
> Threads interact strangely with interrupts: the KeyboardInterrupt exception will be received by an arbitrary thread. (When the signal module is available, interrupts always go to the main thread.)

> Calling sys.exit() or raising the SystemExit exception is equivalent to calling thread.exit().

> It is not possible to interrupt the acquire() method on a lock — the KeyboardInterrupt exception will happen after the lock has been acquired.

python 官方建议使用更高级的threading模块; threading 模块模仿java的线程使用方式，`threading.Thread` 类提供了 `run` 方法供子类重写；并且还提供了更加高级的线程同步机制，包括可重入锁RLock，条件变量 Condition，信号量 Samephore，事件 Event；这些锁的实现稍后我们会对比 java 的 synchronized 和 wait/notify 机制 和 Java 并发包来分析，其实Python 模仿Java实现了这些线程同步机制；更加高级的线程同步锁比如读写锁等，Python 本身并未提供。
## daemon thread/process
### unix daemon process
首先，注意不要把守护线程和UNIX守护进程混淆，这是两个不同的概念。UNIX守护进程也叫daemon，但是他指的是那些脱离终端即不受终端控制的后台进程；UNIX进程中有会话，进程组，进程的概念，一个会话和一个（伪）终端关联，而一个会话包含多个进程组，会话中的进程组分前台进程组和后台进程组，前台进程组会接受来自终端的ctrl+c 等中断信号，但是他们并未脱离终端，只有脱离了终端的后台进程才是守护进程；守护进程主要做一些

### daemon thread
```python
import threading

class DaemonizableThread(threading.Thread):
    def __init__(self, name):
        super(DaemonizableThread, self).__init__(name=name)
        self.count = 0
        self.should_stop = False

    def setDaemon(self, daemonic):
        self.daemon = daemonic

    def run(self):
        while not self.should_stop:
            threading._sleep(2)
            self.count += 2
            print self.name + ' sleeping %s seconds' % self.count

    def stop(self):
        self.should_stop = True

t = DaemonizableThread('demon')
#t.setDaemon(True)

t.start()

# try:
#     t.join()
# except KeyboardInterrupt as e:
#     t.stop()


# import sys
# import signal
# def terminate_handler(sig, frame):
#     print "Caught signal: %s " % sig
#     sys.exit(0)

# signal.signal(signal.SIGTERM, terminate_handler)


try:
    while True:
        t.join(2)
        if not t.isAlive:
            break
except KeyboardInterrupt as e:
    t.stop()

```

## inter thread communication(仿造 inter process communication/IPC)

### thread signal


signal.signal() 函数只能在主线程中使用，即只有主线程可以注册信号处理函数； 任何一个线程都可以使用 alarm(), getsignal(), pause(), setitimer(), getitimer(). 只有主线程可以接受信号，这就是为什么线程通讯不适合用signal。

### thread lock



