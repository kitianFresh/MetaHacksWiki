---
title: "tornado"
date: 2018-01-28 20:27
---

# Tornado 简单用法
## callback 写法
```python
from tornado.httpclient import AsyncHTTPClient
def asynchronous_fetch(url, callback):
    http_client = AsyncHTTPClient()
    def handle_response(response):
        callback(response.body)
    http_client.fetch(url, callback=handle_response)

def print_handler(body):
    print body.text

url = 'http://www.baidu.com'

for i in range(10):
    # 这里并不会阻塞，从而提高并发请求数量，提高吞吐量，服务可用性提高。
    asynchronous_fetch(url, print_handler)
```    

## future 类写法
```python
from tornado.concurrent import Future
import time
def async_fetch_future(url):
    http_client = AsyncHTTPClient()
    my_future = Future()
    fetch_future = http_client.fetch(url)
    fetch_future.add_done_callback(
        lambda f: my_future.set_result(f.result()))
    return my_future

future = async_fetch_future(url)
while not future.done():
    time.sleep(1)
future.result()
```

## 装饰器写法
```python
from tornado import gen

@gen.coroutine
def fetch_coroutine(url):
    http_client = AsyncHTTPClient()
    response = yield http_client.fetch(url)
    raise gen.Return(response.body)
```
Python 2中生成器不允许return操作，因此tornado必须以抛出异常的形式接受返回值。

下面是一个使用 tornado ioloop 的异步TCP 服务器
```python
import errno
import functools
import tornado.ioloop
import socket
import datetime
def connection_ready(sock, fd, events):
    while True:
        try:
            connection, address = sock.accept()
        except socket.error as e:
            if e.args[0] not in (errno.EWOULDBLOCK, errno.EAGAIN):
                raise
            return
        connection.setblocking(0)
        handle_connection(connection, address)

def recover_persistence():
    print 'recover....'
    
if __name__ == '__main__':
    port = 11111
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM, 0)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    sock.setblocking(0)
    sock.bind(("", port))
    sock.listen(9999)

    io_loop = tornado.ioloop.IOLoop.instance()
    # tornado.ioloop.IOLoop.current()
    #Returns the current thread’s IOLoop. 
    # IOLoop是个工厂，一般使用单列模式及 IOLoop.instance() 得到单列，
    #并且这个是主线程中的IOLoop， 其他线程要想得到主线程中的IOLoop就需要使用instance
    callback = functools.partial(connection_ready, sock)
    tornado.ioloop.IOLoop.instance().add_handler(sock.fileno(), callback, io_loop.READ)
    wait_auth = datetime.timedelta(seconds=2)
    tornado.ioloop.IOLoop.instance().add_timeout(wait_auth, recover_persistence)
    tornado.ioloop.IOLoop.instance().start()
```
