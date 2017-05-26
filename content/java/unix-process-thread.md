---
title: "unix-process-thread"
date: 2017-05-24 16:51
---

## Linux下查看 Java/Python 进程线程
先使用 `ps -ef | grep [processname]` 查看进程id
### 进程文件proc
查看status文件; `cat /proc/[pid]/status`;

### pstree
`pstree -p 7604`
```
java(7604)─┬─{java}(7605)
           ├─{java}(7606)
           ├─{java}(7607)
           ├─{java}(7608)
           ├─{java}(7609)
           ├─{java}(7610)
           ├─{java}(7611)
           ├─{java}(7612)
           ├─{java}(7613)
           ├─{java}(7614)
           ├─{java}(7615)
           ├─{java}(7616)
           ├─{java}(7617)
           ├─{java}(7618)
           ├─{java}(7619)
           ├─{java}(7620)
           ├─{java}(7621)
           ├─{java}(7622)
           ├─{java}(7623)
           ├─{java}(7624)
           ├─{java}(7625)
           └─{java}(7626)
```
### top

### ps
`ps -eo ruser,pid,ppid,lwp,psr,args -L | grep java`

## Java 进程线程堆栈分析工具
### jps & jstack
`jps` 一般可以直接用来获取 java 进程 id; 比如我运行了一个8线程的java程序;jps命令格式 `jps [ options ] [ hostid ]`
主选项:

|选    项	|作            用|
|:------:|:--------------:|
|-q	     |只输出LVMID，省略主类的名称|
|-m	     |输出虚拟机进程启动时传递给主类main()函数的参数|
|-l　　   |输出主类的全名，如果进程执行的是jar包，输出jar包路径|
|-v	     |输出虚拟机进程启动时的JVM参数|


```
6993 org.eclipse.equinox.launcher_1.3.200.v20160318-1642.jar
7604 ThreadNums
15721 Jps
1372 Bootstrap
```
进程号是 7604;


jstack（Stack Trace for Java）命令用于生成**虚拟机当前时刻的线程快照。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的目的主要是定位线程长时间出现停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等都是导致线程长时间停顿的原因。线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做些什么事情，或者在等待些什么资源。**

`sudo jstack [pid]` 查看 某个java进程的线程状态. 如果不反应, 使用强制参数 `sudo jstack -F [pid]`;可以直接查到各个线程的状态,运行到哪一句代码等.

```java
Attaching to process ID 7604, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.131-b11
Deadlock Detection:

No deadlocks found.

Thread 7626: (state = IN_JAVA)
 - multicore.forkjoin.PrimeNumberConcurrencyAtomic.run() @bci=21, line=20 (Compiled frame; information may be imprecise)


Thread 7625: (state = IN_JAVA)
 - multicore.forkjoin.PrimeNumberConcurrencyAtomic.run() @bci=21, line=20 (Compiled frame; information may be imprecise)


Thread 7624: (state = IN_JAVA)
 - multicore.forkjoin.PrimeNumberConcurrencyAtomic.run() @bci=21, line=20 (Compiled frame; information may be imprecise)


Thread 7623: (state = IN_JAVA)
 - multicore.forkjoin.PrimeNumberConcurrencyAtomic.run() @bci=21, line=20 (Compiled frame; information may be imprecise)


Thread 7622: (state = IN_JAVA)
 - multicore.forkjoin.PrimeNumberConcurrencyAtomic.run() @bci=21, line=20 (Compiled frame; information may be imprecise)


Thread 7621: (state = IN_JAVA)
 - multicore.forkjoin.PrimeNumberConcurrencyAtomic.run() @bci=21, line=20 (Compiled frame; information may be imprecise)


Thread 7620: (state = IN_JAVA)
 - multicore.forkjoin.PrimeNumberConcurrencyAtomic.run() @bci=21, line=20 (Compiled frame; information may be imprecise)


Thread 7619: (state = IN_JAVA)
 - multicore.forkjoin.PrimeNumberConcurrencyAtomic.run() @bci=21, line=20 (Compiled frame; information may be imprecise)


Thread 7613: (state = BLOCKED)


Thread 7612: (state = BLOCKED)
 - java.lang.Object.wait(long) @bci=0 (Interpreted frame)
 - java.lang.ref.ReferenceQueue.remove(long) @bci=59, line=143 (Interpreted frame)
 - java.lang.ref.ReferenceQueue.remove() @bci=2, line=164 (Interpreted frame)
 - java.lang.ref.Finalizer$FinalizerThread.run() @bci=36, line=209 (Interpreted frame)


Thread 7611: (state = BLOCKED)
 - java.lang.Object.wait(long) @bci=0 (Interpreted frame)
 - java.lang.Object.wait() @bci=2, line=502 (Interpreted frame)
 - java.lang.ref.Reference.tryHandlePending(boolean) @bci=54, line=191 (Interpreted frame)
 - java.lang.ref.Reference$ReferenceHandler.run() @bci=1, line=153 (Interpreted frame)


Thread 7605: (state = BLOCKED)
 - java.lang.Object.wait(long) @bci=0 (Interpreted frame)
 - java.lang.Thread.join(long) @bci=38, line=1252 (Interpreted frame)
 - java.lang.Thread.join() @bci=2, line=1326 (Interpreted frame)
 - multicore.forkjoin.PrimeNumberConcurrencyAtomic.primeNumbers(java.util.concurrent.atomic.AtomicLong, int, long) @bci=65, line=45 (Interpreted frame)
 - multicore.forkjoin.ThreadNums.main(java.lang.String[]) @bci=19, line=12 (Interpreted frame)

```
### jstat
jstat 可以统计虚拟机状态参数信息.命令格式 `jstat [ option vmid [ interval [ s | ms ] [ count ] ] ]`

### jmap
`sudo jmap -heap 7604` 可以查看JVM heap 参数.

## 僵尸进程(zombie)和孤儿进程(orphan)
孤儿进程: 一个**父进程提前退出, 但是子进程还在运行**, 子进程成为孤儿进程. **孤儿进程将被 init 进程(1号进程) 收养, 并由 init 进程对他们完成状态收集.**

僵尸进程: 父进程使用 **fork 创建子进程, 如果子进程先退出**, 而**父进程并没有调用 wait或 waitpid 获取子进程状态信息.**

## 僵尸危害
由于进程退出之后并不是释放所有的资源,内核保留一定的信息,如果进程不调用wait / waitpid的话， 那么保留的那段信息就不会释放，其进程号就会一直被占用，但是系统所能使用的进程号是有限的，如果大量的产生僵死进程，将因为没有可用的进程号而导致系统不能产生新的进程. 此即为僵尸进程的危害，应当避免。


## 查看僵尸进程个数
查看僵尸进程
```
ps -ef | grep defunct | grep -v grep
```
统计僵尸个数
```
ps -ef | grep defunct | grep -v grep | wc -l
```

## 如何杀死僵尸进程
僵尸进程很难用 kill 命令杀死.
### 1.利用信号通信机制
子进程退出时向父进程发送SIGCHILD信号，父进程处理SIGCHILD信号。在信号处理函数中调用wait进行处理僵尸进程。

```C
#include <stdio.h>
#include <unistd.h>
#include <errno.h>
#include <stdlib.h>
#include <signal.h>

static void sig_child(int signo);

int main()
{
    pid_t pid;
    //创建捕捉子进程退出信号
    signal(SIGCHLD,sig_child);
    pid = fork();
    if (pid < 0)
    {
        perror("fork error:");
        exit(1);
    }
    else if (pid == 0)
    {
        printf("I am child process,pid id %d.I am exiting.\n",getpid());
        exit(0);
    }
    printf("I am father process.I will sleep two seconds\n");
    //等待子进程先退出
    sleep(2);
    //输出进程信息
    system("ps -o pid,ppid,state,tty,command");
    printf("father process is exiting.\n");
    return 0;
}

static void sig_child(int signo)
{
     pid_t        pid;
     int        stat;
     //处理僵尸进程
     while ((pid = waitpid(-1, &stat, WNOHANG)) >0)
            printf("child %d terminated.\n", pid);
}
```

## 2. fork 两次
原理是将子进程成为孤儿进程，从而其的父进程变为init进程，通过init进程可以处理僵尸进程

```C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>

int main()
{
    pid_t  pid;
    //创建第一个子进程
    pid = fork();
    if (pid < 0)
    {
        perror("fork error:");
        exit(1);
    }
    //第一个子进程
    else if (pid == 0)
    {
        //子进程再创建子进程
        printf("I am the first child process.pid:%d\tppid:%d\n",getpid(),getppid());
        pid = fork();
        if (pid < 0)
        {
            perror("fork error:");
            exit(1);
        }
        //第一个子进程退出
        else if (pid >0)
        {
            printf("first procee is exited.\n");
            exit(0);
        }
        //第二个子进程
        //睡眠3s保证第一个子进程退出，这样第二个子进程的父亲就是init进程里
        sleep(3);
        printf("I am the second child process.pid: %d\tppid:%d\n",getpid(),getppid());
        exit(0);
    }
    //父进程处理第一个子进程退出
    if (waitpid(pid, NULL, 0) != pid)
    {
        perror("waitepid error:");
        exit(1);
    }
    exit(0);
    return 0;
}
```