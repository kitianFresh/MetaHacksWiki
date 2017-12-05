---
title: "python-thread"
date: 2017-12-05 22:22
---
# python thread

## daemon thread
 
首先，注意不要把守护线程和UNIX守护进程混淆，这是两个不同的概念。UNIX守护进程也叫daemon，但是他指的是哪些脱离的终端即不受终端控制的后台进程；UNIX进程分为会话，进程组，进程，一个会话和一个（伪）终端关联，而一个会话包含多个进程组，会话中的进程组分前台进程组和后台进程组，前台进程组会接受来自终端的ctrl+c 等中断信号，但是他们并未脱离终端，只有脱离了终端的后台进程才是守护进程；
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

## inter thread communication

### thread signal


signal.signal() 函数只能在主线程中使用，即只有主线程可以注册信号处理函数； 任何一个线程都可以使用 alarm(), getsignal(), pause(), setitimer(), getitimer(). 只有主线程可以接受信号，这就是为什么线程通讯不适合用signal。

### thread lock



