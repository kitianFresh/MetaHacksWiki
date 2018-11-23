---
title: "python-shared-mem"
date: 2018-05-22 21:43
---

# 多进程之间共享一份数据
## 控制信号
### 信号机制
### 同步互斥锁信号量

## 非结构化数据（字节数组）
### UNIX 匿名管道/有名管道FIFO

## 结构化数据
### 消息队列

## 大数据量
### 通过队列共享
### 通过文件共享（普通文件，或者tmpfs很快/dev/shm）
### 通过内存共享（MMAP）
我们如何共享一个很大的 `numpy.ndarray` 数组呢？
1. 第一种方式是直接使用 `np.memmap` 这种方式读取数据。
   - 这其实没有把数据全部读入内存，而是通过对文件流进行封装. 其实是一种共享文件的方式。

2. 第二种方式，我们可以使用 `pickle/unpickle` 的方式。
   - 这种其实是在复制数据，而且解析/反解析很费时间。`multiprocessing.Pool` 就是这种方式，内存数据还是多份的。

3. 第三种方式，我们可以使用 `mmap` 映射数组。
   - 这种方式

4. 第四种方式，ctypes + fork-copy-on-write 机制。
  - 这种方法对于只读数据是有效的，可以把共享的numpy数组放在进程的全局变量当中，但是不要修改他，利用fork的写时复制特性，只要fork的子进程只读数据，就可以防止数据被复制。可以避免使用 pickle/unpickle解析/反解析数据。
> create the shared array as a ctypes object, and pass it into the initializer of each process, making it a global variable in each process.
```python
# <!-- 包装 -->
from multiprocessing import sharedctypes
size = S.size
shape = S.shape
S.shape = size
S_ctypes = sharedctypes.RawArray('d', S)
S = numpy.frombuffer(S_ctypes, dtype=numpy.float64, count=size)
S.shape = shape

# <!-- 拿到可用数组 -->
from numpy import ctypeslib
S = ctypeslib.as_array(S_ctypes)
S.shape = shape
```
需要使用ctypes把numpy数组包装起来，否则不能直接使用。

```python
from multiprocessing import Pool, sharedctypes
import numpy as np
import warnings

def populate(args):
    """ Populates the portion of the shared array arr indicated by the offset and blocksize options """
    offset, blocksize = args
    with warnings.catch_warnings():
        warnings.simplefilter('ignore', RuntimeWarning)
        v = np.ctypeslib.as_array(arr)

    for idx in range(offset, blocksize+offset):
        v[idx, :] = offset

    return offset

def _init(arr_to_populate):
    """ Each pool process calls this initializer. Load the array to be populated into that process's global namespace """
    global arr
    arr = arr_to_populate

size = 2000
blocksize = 100
tmp = np.ctypeslib.as_ctypes(np.zeros((size, 10)))
shared_arr = sharedctypes.Array(tmp._type_, tmp, lock=False) 

pool = Pool(processes=4, initializer=_init, initargs=(shared_arr, ))
result = pool.map(populate, zip(range(0, size, blocksize), [blocksize]*(size/blocksize)))

with warnings.catch_warnings():
    warnings.simplefilter('ignore', RuntimeWarning)
    final_arr = np.ctypeslib.as_array(shared_arr)

print final_arr
```

# Mac 和 Ubuntu 系统对numpy数组分配的区别
## Mac 系统和 Linux系统内存管理方式
同样一份代码和数据，机器物理内存配置都是一样的，Mac上运行良好，但是在 Ubuntu 上却频繁出现 MemoryError. 我使用的是Google Cloud Platform VM 实例，15G 内存还是爆炸了。这是为什么，后来发现，GCP VM 默认没有Swap分区的，没有内存页面和磁盘的交换。

> OSX machine is paging RAM onto disk allowing you to have more data in memory that is in physical RAM. The EC2 instance probably doesn't have swap set up since a system and a volume on EC2 are different concepts.

添加交换区并开启, 交换区大小一般可以设置为物理内存的1到2倍。

```sh
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo swapon -s
```
可以在文件 `/etc/fstab` 中添加一行 `/swapfile none swap sw 0 0`. 可以在系统启动的时候自动挂载并开启交换区。如果需要扩容交换区，
```sh
sudo swapoff /swapfile 
sudo fallocate -l 8G /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```