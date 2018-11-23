---
title: "python-exception"
date: 2017-12-07 23:22
---

```python
def div(a,b):
    return a/b

def algo(a,b):
    return div(1,b) + 1

try:
    algo(1,0)
except:
    import traceback, StringIO, re, os

    def colorize(text, color):
        control_sequence_introducer = "\x1B["
        return "{0}{1}m{2}{0}0m".format(control_sequence_introducer, color, text)

    exception_output = StringIO.StringIO()
    traceback.print_exc(file=exception_output)
    stack_trace = exception_output.getvalue().splitlines()
    print stack_trace
    stack_trace.pop(0)
    error = stack_trace.pop()

    print ''

    i = 0
    for line in stack_trace:
        line = line.strip()
        if 0 == i % 2:
            matches = re.match('File "(.*)", line (\d+), in (.+)', line)
            if matches:
                groups = matches.groups()
                path = groups[0]
                path = path.replace(os.getcwd(), '')
                line_number = groups[1]
                method = groups[2]
        else:
            ident =' '
            print ident * i + "{0} ({1}): {2}: {3}".format(
                colorize(path, 36),
                colorize(line_number, 33),
                colorize(method, 32),
                colorize(line, 31)
            )
        i = i + 1

    print colorize("Error ", 33) + colorize(error, 31)
    print ''
```

# colorize
```python
def colorize(text, color):
        control_sequence_introducer = "\x1B["
        return "{0}{1}m{2}{0}0m".format(control_sequence_introducer, color, text)

print repr(colorize('fuck', 31))
print colorize('fuck', 32)
print colorize('fuck', 33)
print colorize('fuck', 34)
print colorize('fuck', 35)
print colorize('fuck', 36)
print colorize('fuck', 37)
print colorize('fuck', 38)
```

```python
def print_format_table():
    """
    prints table of formatted text format options
    """
    for style in range(8):
        for fg in range(30,38):
            s1 = ''
            for bg in range(40,48):
                format = ';'.join([str(style), str(fg), str(bg)])
                s1 += '\x1b[%sm %s \x1b[0m' % (format, format)
            print(s1)
        print('\n')

print_format_table()

```


## thread exception dump stack

```python
import thread
import threading
import sys
import traceback
import logging
from tornado.ioloop import IOLoop


def is_main_thread():
    cur_tid = thread.get_ident()
    main_tid = None
    main_inst = IOLoop.instance()
    if main_inst is not None:
        main_tid = main_inst._thread_ident
    if main_tid is None:
        main_tid = cur_tid
    if cur_tid != main_tid:
        return False
    else:
        return True


# class safe_acquire_lock(object):
#    def __init__(self, lock):
#        self.lock = lock
#    def __enter__(self):
#        self.lock.acquire()
#    def __exit__(self, type, value, traceback):
#        self.lock.release()


def get_thread_id():
    th = threading.current_thread()
    return '%d-%s' % (th.ident, th.name)


def dump_thread_stacks():
    frames = sys._current_frames()
    for t in threading.enumerate():
        if t.ident is not None:
            f = frames.get(t.ident)
            stack = traceback.extract_stack(f)
            logging.info('%s(%s-%d): %s', t, t.__class__.__name__, t.ident, stack)


def get_object_in_stack(test_func):
    frame = sys._getframe()
    while frame.f_back is not None:
        for v in frame.f_locals.values():
            if test_func(v):
                return v
        frame = frame.f_back
    return None


def get_object_top_call_in_stack(obj, excludes=[]):
    calls = []
    for f in dir(obj):
        a = getattr(obj, f) 
        if callable(a) and \
                (not isinstance(excludes, list) or f not in excludes):
            calls.append(f)
    frame = sys._getframe().f_back
    top_call = None
    while frame is not None:
        if obj in frame.f_locals.values() and \
                frame.f_code.co_name in calls:
            top_call = frame.f_code.co_name
        frame = frame.f_back
    return top_call


def get_object_of_classes_in_stack(cls_list, excludes=None):
    def _is_instance(v):
        for cls in cls_list:
            if isinstance(v, cls) and \
                    (excludes is None or v not in excludes):
                return True
        return False
    return get_object_in_stack(_is_instance)


if __name__ == '__main__':
    logging.getLogger().setLevel(logging.INFO)

    class Test(object):
        def call(self):
            dump_thread_stacks()
#             print get_object_of_classes_in_stack((Test,))
#             print get_object_top_call_in_stack(self)
    Test().call()


class worker_thread(object):

    def __init__(self, func):
        self.__func = func

    def __call__(self, *args, **kwargs):
        if self.__func:
            if is_main_thread():
                thread.start_new(lambda: self.__do_func(*args, **kwargs), ())
            else:
                self.__do_func(*args, **kwargs)

    def __do_func(self, *args, **kwargs):
        try:
            self.__func(*args, **kwargs)
        except Exception as e:
            logging.error(e)


def assert_main_thread_invoke(name='invoke'):
    if is_main_thread():
        import traceback
        stack = traceback.format_stack()
        s_stack = ''.join(stack)
        logging.warning('%s on main thread: %s' % (name, s_stack))

```