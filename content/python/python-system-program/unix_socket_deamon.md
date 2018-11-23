---
title: "unix_socket_deamon"
date: 2017-09-08 10:33
---

# 本地进程之间通过实REST来进行进程通讯
unix socket 可以提供本机的不同进程间通过类似于 http 的请求来通讯；后台进程服务启动入口，`init_deamon_process`
```python
# -*- coding: utf-8 -*-
import os
import socket
import json
import subprocess
import logging
from tornado.options import options
from tornado.httpclient import HTTPError as ClientHTTPError
from clouds.common import codeutils
from clouds.common.tornadoutils import cerror2serror


def is_service_on(unix_socket):
    if not os.path.exists(unix_socket):
        return False
    s = socket.socket(socket.AF_UNIX)
    try:
        s.connect(unix_socket)
    except socket.error:
        return False
    else:
        return True
    finally:
        s.close()


_region_mebsd = None
_workspace = None
_unix_socket = None


def register_deamon_task(id, action, zone, region_task_id=None):
    from clouds.common.http_uds_client import get_client
    client = get_client()
    url = '/'
    data = {'id': id, 'action': action, 'zone': zone}
    if region_task_id is not None:
        data['region_task_id'] = region_task_id
    body = json.dumps(data)
    try:
        client.sync_request(_unix_socket, 'POST', url, body=body)
    except ClientHTTPError as e:
        serr = cerror2serror(e)
        logging.error('Register deamon long time task failed: %s' % str(serr))
        raise serr


def init_deamon_process():
    global _deamon_process, _workspace, _unix_socket
    _workspace = os.path.join(codeutils.get_workspace_path(), 'task_deamon')
    if not os.path.exists(_workspace):
        os.mkdir(_workspace)
    _unix_socket = os.path.join(_workspace, 'task_deamon.socket')
    if is_service_on(_unix_socket):
        logging.info('task_deamon is running...')
    else:
        logging.info('start to run region_mebsd...')
        _deamon_process = subprocess.Popen(
            [
                "./task_deamon",
                "--workspace=%s" % _workspace,
                "--unix-socket=%s" % _unix_socket,
                "--config-file=%s" % options.config_file,
            ], close_fds=True)                              # close_fds doesn't apply to fd 0,1,2


def stop_deamon_process():
    global _deamon_process
    if _deamon_process is not None:
        _deamon_process.terminate()   # send signal TERM

```

和后台进程通讯的 unix socket 客户端
```python
from clouds.common.httputils import AsyncHTTPClientManager
import urllib
import httplib
import socket
import sys


class HttpUnixDomainSocketClient(AsyncHTTPClientManager):

    def sync_request(self, socket_path, method, url, *args, **kwargs):
        quote_path = urllib.quote(socket_path, safe='')
        if not url.startswith('/'):
            url = '/' + url
        req_url = 'http+unix://' + quote_path + url
        kwargs['sync'] = True
        return self.json_fetch(method, req_url, *args, **kwargs)


_default_client = None


def get_client():
    global _default_client
    if _default_client is None:
        print "Http unix domain socket client initialized"
        _default_client = HttpUnixDomainSocketClient()
    return _default_client


def _has_timeout(timeout):
    if hasattr(socket, '_GLOBAL_DEFAULT_TIMEOUT'):
        return timeout is not None and timeout is not socket._GLOBAL_DEFAULT_TIMEOUT
    return timeout is not None


class HttpUnixDomainSocketConnection(httplib.HTTPConnection):
    """
    HTTP over UNIX Domain Sockets
    URI format: http+unix://%2Fpath%2Fto%2Fsocket.sock/location
    Socket path is url_quoted
    """

    def __init__(self, host, port=None, strict=None, timeout=None, proxy_info=None):
        httplib.HTTPConnection.__init__(self, host, port)
        self.timeout = timeout

    def connect(self):

        try:
            self.sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)

            if _has_timeout(self.timeout):
                self.sock.settimeout(self.timeout)

            self.sock.connect(urllib.unquote(self.host))
        except socket.error as msg:
            if self.sock:
                self.sock.close()
            self.sock = None

            raise socket.error(msg)


# Monkey patch this module to httplib2
sys.modules['httplib2'].SCHEME_TO_CONNECTION['http+unix'] = HttpUnixDomainSocketConnection

```

后台耗时任务进程, 采用的是 tornado 异步框架，相当于一个 web 后台服务；后台耗时任务完成之后，再通过 REST 风格的请求回调回去任务的注册者，告知任务注册者任务执行完毕，也就是说对于任务的注册者，任务的进行时异步的，这种异步是基于 REST 实现而不是 RPC或者消息。
```python
# import time
import os
import json
import urlparse
import sys
import tornado
import Queue
import threading
import signal
import datetime
import functools

from tornado.httpclient import AsyncHTTPClient, HTTPRequest
from tornado.httpserver import HTTPServer
from tornado.httputil import HTTPHeaders
from tornado.ioloop import IOLoop
from tornado.netutil import bind_unix_socket
from tornado.web import RequestHandler, Application, HTTPError
from tornado.gen import coroutine
from tornado.options import define, options
from clouds.mebs import client as mebs_client
from clouds.common import config
from clouds.common import auth
from clouds.common.handler.ping import PingHandler
from clouds.common.service import ServiceBase

from clouds.models.disks import Disks
from clouds.models.mebs_backups import MEBSBackups


NUM_WORKERS = 4
CHECK_INTERVAL = 10
TIMEOUT = 3600 * 12
PERSISTENCE_FILE = 'data.keep'

queue = Queue.Queue()
persistence = set()

ACTIONS = ['clone_snapshot', 'clone', 'backup', 'restore_backup', 'delete_backup',
           'create_template', 'commit_template']


class Task(object):

    def __init__(self, id, action, zone):
        """ Note: backup task doesn't know its size when calling the API,
            needs to report here in region_mebsd
        """
        self.id = id
        self.action = action
        self.zone = zone    # zone name
        self.count = 0
        self.size_in_bytes = 0
        self.size_reported = True
        self.region_task_id = None

    def __str__(self):
        return 'mebsd task => id=%s, action=%s, zone=%s, count=%s' % (self.id, self.action, self.zone, self.count)

    def execute(self):
        self.count += CHECK_INTERVAL
        finished, error = self.get_task_status()
        if error:
            IOLoop.instance().add_callback(MainHandler.task_failed, self)
        elif finished:
            IOLoop.instance().add_callback(MainHandler.task_finished, self)
        elif self.count < TIMEOUT:
            IOLoop.instance().add_callback(MainHandler.check_task_later, self)
        else:
            IOLoop.instance().add_callback(MainHandler.task_failed, self)

    def get_task_status(self):
        from clouds.mebs.client import get_client
        mebs_cli = get_client(self.zone)
        if self.action == 'clone_snapshot' or self.action == 'clone':
            p = mebs_cli.qcow2.Snapshot.get_clone_progress(self.id)
            if p is None:
                return False, False
            finished = True if p.percent == 100 else False
            return finished, p.error

        elif self.action == 'backup':
            p = mebs_cli.qcow2.Snapshot.get_backup_progress(self.id)
            if p is None:
                return False, False
            if p.total_size_byte > 0 and self.size_in_bytes != p.total_size_byte:
                self.size_in_bytes = p.total_size_byte
                self.size_reported = False
            finished = True if p.percent == 100 else False
            return finished, p.error

        elif self.action == 'restore_backup':
            p = mebs_cli.qcow2.Snapshot.get_restore_progress(self.id)
            if p is None:
                return False, False
            finished = True if p.percent == 100 else False
            return finished, p.error

        elif self.action == 'delete_backup':
            p = mebs_cli.qcow2.Snapshot.get_delbackup_progress(self.id)
            if p is None:
                return False, False
            finished = True if p.percent == 100 else False
            return finished, p.error

        elif self.action == 'create_template':
            p = mebs_cli.template.get_create_progress(self.id)
            if p is None:
                return False, False
            finished = True if p.percent == 100 else False
            return finished, p.error

        elif self.action == 'commit_template':
            p = mebs_cli.template.get_commit_progress(self.id)
            if p is None:
                return False, False
            finished = True if p.percent == 100 else False
            return finished, p.error

        elif self.action == 'migrate_template':
            p = mebs_cli.qcow2.query_migrate(self.id)
            if p is None:
                return False, False
            finished = True if p.percent == 100 else False
            return finished, p.error

        return False, True


class WorkerManager(object):

    def init(self):
        for i in range(NUM_WORKERS):
            t = threading.Thread(target=self.worker)
            t.setDaemon(True)
            t.start()

    @staticmethod
    def worker():
        while True:
            task = queue.get()
            task.execute()
            queue.task_done()


class MainHandler(RequestHandler):

    @coroutine
    def post(self, *args, **kwargs):
        """ json body example:
            {'action': 'action', 'id': 'idstr', 'zone': 'zone'...}
        """
        try:
            body = json.loads(self.request.body)
        except Exception:
            raise HTTPError(400, 'Invalid body')
        if body['action'] not in ACTIONS:
            raise HTTPError(400, 'Invalid action')
        task = Task(body['id'], body['action'], body['zone'])
        if 'region_task_id' in body:
            task.region_task_id = body['region_task_id']
        queue.put(task)
        print task, 'start'
        self.write(json.dumps(''))

    @classmethod
    @coroutine
    def json_fetch(cls, method, url, headers=None, data=None):
        service_url = auth.get_manager().get_service_url('compute', region=options.region)
        token = auth.get_manager().get_token()
        url = urlparse.urljoin(service_url, url)
        client = AsyncHTTPClient()
        xheaders = {'X-Auth-Token': token,
                    "Content-Type": "application/json charset=utf-8"}
        if headers is not None:
            xheaders.update(headers)
        xheaders = HTTPHeaders(xheaders)
        if data is None:
            data = {}
        body = json.dumps(data)
        request = HTTPRequest(url, method, xheaders, body)
        response = yield client.fetch(request)
        raise tornado.gen.Return(response)

    @classmethod
    @coroutine
    def check_task_later(cls, task):

        def insert(_task):
            persistence.remove(_task)
            queue.put(_task)

        persistence.add(task)
        if task.action == 'backup' and not task.size_reported:
            task.size_reported = True       # only report to region once
            data = {'mebs_backup': {'size': task.size_in_bytes / 1024 / 1024}}
            yield cls.json_fetch('PUT', '/mebs_backups/%s' % task.id, data=data)

        IOLoop.instance().add_timeout(datetime.timedelta(seconds=CHECK_INTERVAL),
                                      functools.partial(insert, task))

    @classmethod
    @coroutine
    def task_finished(cls, task):
        print task, 'finished'
        if task.action == 'clone_snapshot':
            data = {'disk': {'status': Disks.DISK_READY}}
            yield cls.json_fetch('PUT', '/disks/%s' % task.id, data=data)

        elif task.action == 'restore_backup':
            yield cls.json_fetch('POST', '/disks/%s/notify-restore-mebs-backup-succ' % task.id,
                                 data={'disk': {}})

        elif task.action == 'backup':
            data = {'mebs_backup': {'status': MEBSBackups.STATUS_READY}}
            if not task.size_reported:
                data['mebs_backup']['size'] = task.size_in_bytes / 1024 / 1024
            yield cls.json_fetch('PUT', '/mebs_backups/%s' % task.id, data=data)

        elif task.action == 'delete_backup':
            yield cls.json_fetch('POST', '/mebs_backups/%s/notify-delete-succ' % task.id,
                                 data={'mebs_backup': {}})

        if task.region_task_id is not None:
            yield cls.json_fetch('POST', '/tasks/%s' % task.region_task_id)

    @classmethod
    @coroutine
    def task_failed(cls, task):
        print task, 'failed'
        if task.action == 'clone_snapshot':
            yield cls.json_fetch('PUT', '/disks/%s' % task.id,
                                 data={'disk': {'status': Disks.CLONE_MEBS_SNAPSHOT_FAILED}})

        elif task.action == 'restore_backup':
            yield cls.json_fetch('POST', '/disks/%s/notify-restore-mebs-backup-fail' % task.id)

        elif task.action == 'backup':
            yield cls.json_fetch('PUT', '/mebs_backups/%s' % task.id,
                                 data={'mebs_backup': {'status': MEBSBackups.STATUS_BACKUP_FAILED}})

        elif task.action == 'delete_backup':
            yield cls.json_fetch('POST', '/mebs_backups/%s/notify-delete-fail' % task.id)

        if task.region_task_id is not None:
            yield cls.json_fetch('POST', '/tasks/%s' % task.region_task_id,
                                 data={'__status__': 'error', '__reason__': 'region_mebsd task failed'})


def receive_signal(signum, stack):

    def persistence_then_exit():
        data = []
        make_data = lambda t: {'id': t.id, 'action': t.action, 'zone': t.zone,
                               'count': t.count, 'size_in_bytes': t.size_in_bytes,
                               'size_reported': t.size_reported,
                               'region_task_id': t.region_task_id}
        while not queue.empty():
            task = queue.get()
            data.append(make_data(task))
        for task in persistence:
            data.append(make_data(task))
        if data:
            with open(os.path.join(options.workspace, PERSISTENCE_FILE), 'w') as f:
                f.write(json.dumps(data))
        sys.exit(0)     # exit the main thread, the other daemon worker threads exit as well

    if signum in ServiceBase.get_trap_signums():
        IOLoop.instance().add_callback_from_signal(persistence_then_exit)


def trap_signals():
    for signum in ServiceBase.get_trap_signums():
        signal.signal(signum, receive_signal)


def recover_persistence():
    filepath = os.path.join(options.workspace, PERSISTENCE_FILE)
    if not os.path.exists(filepath):
        return
    with open(filepath, 'r') as f:
        data = json.loads(f.read())
    for i in data:
        task = Task(i['id'], i['action'], i['zone'])
        task.count = i['count']
        task.size_in_bytes = i['size_in_bytes']
        task.size_reported = i['size_reported']
        task.region_task_id = i['region_task_id']
        queue.put(task)
    os.unlink(filepath)


def main():
    define('workspace', help='Workspace dir')
    define('unix_socket', help='Unix Domain Socket file')
    define('enable_mebs', default=False, type=bool, help='MEBS service trigger')
    define('mebs_src_dir', help='MEBS API source code dir')
    define('mebs_bin_dir', help='MEBS binary exe dir')
    define('mebs_log_config', default='/etc/mebs/log.conf', help='Logging level control')
    define('mebs_log_dir', help='MEBS log dir')
    define('mebs_manager_config')
    define('mebs_redis_config')
    define('mebs_sql_connection')

    config.init("etc/region.conf")

    config.check_options(['region', 'port', 'auth_uri',
                          'admin_user', 'admin_password',
                          'admin_tenant_name', 'workspace', 'unix_socket'])

    if getattr(options, 'enable_mebs', False):
        config.check_options(['mebs_src_dir', 'mebs_redis_config'])
        mebs_client.configure_at_region()

    wait_auth = datetime.timedelta(seconds=2)
    IOLoop.instance().add_timeout(wait_auth, recover_persistence)
    trap_signals()
    auth.init()
    worker_man = WorkerManager()
    worker_man.init()

    application = Application([(r"/ping", PingHandler),
                               (r"/", MainHandler)])
    server = HTTPServer(application, xheaders=True)
    sock = bind_unix_socket(options.unix_socket)
    server.add_socket(sock)
    server.start()
    IOLoop.instance().start()

```