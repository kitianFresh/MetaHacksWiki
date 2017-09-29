---
title: "python-process"
date: 2017-09-29 19:43
---
# subprocess
subprocess 和 multiprocessing
 - subprocess 是让进程可以执行一个子程序进程，类似 fork + exec 的模式。我们知道，比较底层的创建进程的接口是fork，然后代码逻辑通过fork的一次调用两次返回，通过父子进程返回的值不同来区分控制父子进程代码的执行逻辑，典型的就是 if else 的判断。如果在子进程的逻辑代码中有可以复用的部分，可以把这部分做成一个子程序，便于其他进程直接使用，可以直接让子进程调用 exec 函数来执行一个可执行程序。

 - multiprocessing更像是和threading模块一样，提供直接传入函数开启一个进程或者线程的功能。

Python 官方文档建议使用 subprocess 替代一下模块
- os.system
- os.spawn*
- os.popen*
- popen2.*
- commands.*

## check_call
以下都是阻塞的调用。
1. subprocess.call(args, *, stdin=None, stdout=None, stderr=None, shell=False)
  - 执行cmd，并返回一个值，不抛出异常; ret != 0 表示程序返回值非0即程序异常退出。

2. subprocess.check_call(args, *, stdin=None, stdout=None, stderr=None, shell=False)
  - 执行cmd命令，等待执行完成，然后返回，如果执行出错即异常退出，则抛出异常。

3. subprocess.check_output(args, *, stdin=None, stderr=None, shell=False, universal_newlines=False)
  - 调用子程序并以byte string 返回子程序的输出；
  - 调用失败抛出 CalledProcessError, 并且Error含有两个属性，returncode 和 output
  
>Note Do not use stderr=PIPE with this function as that can deadlock based on the child process error volume. Use Popen with the communicate() method when you need a stderr pipe.

shell 参数表示是否使用shell来执行命令。一般情况下不使用，除非该命令是shell的内置命令，只能由shell来执行。

```python
subprocess.check_call(["ls", "-l"])

ret = subprocess.call("exit 2", shell=True)
if ret != 0:
    print "failed!"
else:
    print "succ"
```

```python
try:
    subprocess.check_call("exit 1", shell=True)
except subprocess.CalledProcessError as e:
    print e.returncode
    print e.output
    print e.cmd
    #print ret 此时ret还未赋值
```

```python
returned_string = None
try:
    returned_string = subprocess.check_output("exit 2", shell=True)
except subprocess.CalledProcessError as e:
    print e.returncode
    print e.output
    print e.cmd
if returned_string is not None:
    print returned_string


subprocess.check_output("ls non_existent_file; exit 0",
                   stderr=subprocess.STDOUT,
                   shell=True)
```
## subprocess.Popen
以上接口都是通过更加底层的 subprocess.Popen()来实现的。这个函数在linux上就是 fork + execv 实现的，windows 上则是通过 CreateProcess() 实现的。
4. class subprocess.Popen(args, bufsize=0, executable=None, stdin=None, stdout=None, stderr=None, preexec_fn=None, close_fds=False, shell=False, cwd=None, env=None, universal_newlines=False, startupinfo=None, creationflags=0)
  - args参数是一个list或者字符串儿，如果是list，就执行list的第一项元素即可执行程序，后续的项是可执行程序的参数；如果是字符串，默认整个串儿就是可执行程序，所以一般在命令行中使用 shlex.
  - args：要执行的命令或可执行文件的路径。一个由字符串组成的序列（通常是列表），列表的第一个元素是可执行程序的路径，剩下的是传给这个程序的参数，如果没有要传给这个程序的参数，args 参数可以仅仅是一个字符串。
  - bufsize：控制 stdin, stdout, stderr 等参数指定的文件的缓冲，和打开文件的 open()函数中的参数 bufsize 含义相同。
  - executable：如果这个参数不是 None，将替代参数 args 作为可执行程序；
  - stdin：指定子进程的标准输入；
  - stdout：指定子进程的标准输出；
  - stderr：指定子进程的标准错误输出；
　　对于 stdin, stdout 和 stderr 而言，如果他们是 None（默认情况），那么子进程使用和父进程相同的标准流文件。

　　父进程如果想要和子进程通过 communicate() 方法通信，对应的参数必须是 subprocess.PIPE（见下文例4）；

　　当然 stdin, stdout 和 stderr 也可以是已经打开的 file 对象，前提是以合理的方式打开，比如 stdin 对应的文件必须要可读等。　

  - preexec_fn：默认是None，否则必须是一个函数或者可调用对象，在子进程中首先执行这个函数，然后再去执行为子进程指定的程序或Shell。
  - close_fds：布尔型变量，为 True 时，在子进程执行前强制关闭所有除 stdin，stdout和stderr外的文件；
  - shell：布尔型变量，明确要求使用shell运行程序，与参数 executable 一同指定子进程运行在什么 Shell 中——如果executable=None 而 shell=True，则使用 /bin/sh 来执行 args 指定的程序；也就是说，Python首先起一个shell，再用这个shell来解释指定运行的命令。
  - cwd：代表路径的字符串，指定子进程运行的工作目录，要求这个目录必须存在；
  - env：字典，键和值都是为子进程定义环境变量的字符串；
  - universal_newline：布尔型变量，为 True 时，stdout 和 stderr 以通用换行（universal newline）模式打开，
  - startupinfo：见下一个参数；
  - creationfalgs：最后这两个参数是Windows中才有的参数，传递给Win32的CreateProcess API调用。

进程对象属性：
1. p.returncode 该属性表示子进程的返回状态，returncode可能有多重情况：

 - None —— 子进程尚未结束；
 - ==0 —— 子进程正常退出；
 - \> 0—— 子进程异常退出，returncode对应于出错码；
 - < 0—— 子进程被信号杀掉了。

2. p.stdin, p.stdout, p.stderr, p.pid

进程对象方法：
1. p.poll()
  - 检查子进程 p 是否终止，返回的而是 p.returncode

2. p.wait()
  - 父进程会阻塞等待，直到子进程结束
  
3. p.communicate(input=None)
  - 和子进程 p 交流，将参数 input （字符串）中的数据发送到子进程的 stdin，同时从子进程的 stdout 和 stderr 读取数据，直到EOF。

　- 返回值：二元组 (stdoutdata, stderrdata) 分别表示从标准出和标准错误中读出的数据。

　　父进程调用 p.communicate() 和子进程通信有以下限制：

　　（1） 只能通过管道和子进程通信，也就是说，只有调用 Popen() 创建子进程的时候传入是参数 stdin=subprocess.PIPE，才能通过 p.communicate(input) 向子进程的 stdin 发送数据；只有参数 stout 和 stderr 也都为 subprocess.PIPE ，才能通过p.communicate() 从子进程接收数据，否则接收到的二元组中，对应的位置是None。

　　（2）父进程从子进程读到的数据缓存在内存中，因此commucate()不适合与子进程交换过大的数据。

　　注意：

　　　　communicate() 立即阻塞父进程，直到子进程结束！

4. p.send_signal(signal)
  - 向子进程发送信号

5. p.terminate()
  - 终止子进程，等价于 p.send_signal(SIGTERM)

6. p.kill()
  - 杀死子进程，等价于 p.send_signal(SIGKILL)


```python
import shlex, subprocess
# 'cat tmux-client-4152.log > /Users/kiki/study/test/echo.txt'
# 以上命令不能直接用在Popen中，因为这样他只会默认读取第一项是cat命令执行，后面都是cat的参数。读到 `>` 就不认识了。
command_line = raw_input()
args = shlex.split(command_line)
print args
p = subprocess.Popen(args)
# tmux-client-4152.log
```

pipeline
```python
import subprocess
#方案一
c1 = subprocess.Popen(['ls', '-alt'], stdout=subprocess.PIPE)
#以下两种形式注意对比区别（1）
# c2 = subprocess.Popen(['wc', '-w'], stdin=c1.stdout, stdout=subprocess.PIPE)
c2 = subprocess.Popen(['wc', '-w'], stdin=c1.stdout, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
stdout, stderr = c2.stdout, c2.stderr
for line in stdout:
    print line
for line in stderr:
    print line
#stdout, stderr = c2.communicate()
c1.wait() # 调用wait 防止父进程比子进程提前结束
c2.wait()
print (stdout, stderr)

# 方案二
print subprocess.check_output("ls -alt | wc -w", shell=True)
```


## Process Groups / Sessions¶
通过subprocess 得到子进程，可以通过 os.kill 来向子进程发送信号；
```python
import os
import signal
import subprocess
import tempfile
import time
import sys

script = '''#!/bin/sh
echo "Shell script in process $$"
set -x
python signal_child.py
'''
script_file = tempfile.NamedTemporaryFile('wt')
script_file.write(script)
script_file.flush()

proc = subprocess.Popen(['sh', script_file.name], close_fds=True)
print 'PARENT      : Pausing before sending signal to child %s...' % proc.pid
sys.stdout.flush()
time.sleep(1)
print 'PARENT      : Signaling child %s' % proc.pid
sys.stdout.flush()
os.kill(proc.pid, signal.SIGUSR1)
time.sleep(3)
```

但是对于子进程又创建的孙子进程，孙子进程是无法接受到信号的，必须通过进程组合回话来控制！ 也就是必须让孙子进程在子进程的回话组中！ 才能够接受到父进程发送的信号！ os.killpg()

```python
import os
import signal
import subprocess
import tempfile
import time
import sys

script = '''#!/bin/sh
echo "Shell script in process $$"
set -x
python signal_child.py
'''
script_file = tempfile.NamedTemporaryFile('wt')
script_file.write(script)
script_file.flush()

proc = subprocess.Popen(['sh', script_file.name], 
                        close_fds=True,
                        preexec_fn=os.setsid,
                        )
print 'PARENT      : Pausing before sending signal to child %s...' % proc.pid
sys.stdout.flush()
time.sleep(1)
print 'PARENT      : Signaling process group %s' % proc.pid
sys.stdout.flush()
os.killpg(proc.pid, signal.SIGUSR1)
time.sleep(3)
```

The sequence of events is:

The parent program instantiates Popen.
The Popen instance forks a new process.
The new process runs os.setsid().
The new process runs exec() to start the shell.
The shell runs the shell script.
The shell script forks again and that process execs Python.
Python runs signal_child.py.
The parent program signals the process group using the pid of the shell.
The shell and Python processes receive the signal. The shell ignores it. Python invokes the signal handler.

subprocess模块的缺陷在于默认提供的父子进程间通信手段有限，只有管道；同时创建的子进程专门用来执行外部的程序或命令。

## 参考
- [subprocess – Work with additional processes](https://pymotw.com/2/subprocess/)