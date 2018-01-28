---
title: "python-logging"
date: 2017-08-23 11:32
---

# 关于tornado的疑惑
## 为什么运行在tornado之上的webapp可以同时把日志输出到控制台和日志文件
tornado 的日志模块有一个默认日志配置函数，有关于日志配置选项。这个函数是在 tornado.options 模块中被全局初始化调用的；
```python
def define_logging_options(options=None):
    """Add logging-related flags to ``options``.

    These options are present automatically on the default options instance;
    this method is only necessary if you have created your own `.OptionParser`.

    .. versionadded:: 4.2
        This function existed in prior versions but was broken and undocumented until 4.2.
    """
    if options is None:
        # late import to prevent cycle
        import tornado.options
        options = tornado.options.options
    options.define("logging", default="info",
                   help=("Set the Python log level. If 'none', tornado won't touch the "
                         "logging configuration."),
                   metavar="debug|info|warning|error|none")
    options.define("log_to_stderr", type=bool, default=None,
                   help=("Send log output to stderr (colorized if possible). "
                         "By default use stderr if --log_file_prefix is not set and "
                         "no other logging is configured."))
    options.define("log_file_prefix", type=str, default=None, metavar="PATH",
                   help=("Path prefix for log files. "
                         "Note that if you are running multiple tornado processes, "
                         "log_file_prefix must be different for each of them (e.g. "
                         "include the port number)"))
    options.define("log_file_max_size", type=int, default=100 * 1000 * 1000,
                   help="max size of log files before rollover")
    options.define("log_file_num_backups", type=int, default=10,
                   help="number of log files to keep")

    options.define("log_rotate_when", type=str, default='midnight',
                   help=("specify the type of TimedRotatingFileHandler interval "
                         "other options:('S', 'M', 'H', 'D', 'W0'-'W6')"))
    options.define("log_rotate_interval", type=int, default=1,
                   help="The interval value of timed rotating")

    options.define("log_rotate_mode", type=str, default='size',
                   help="The mode of rotating files(time or size)")

    options.add_parse_callback(lambda: enable_pretty_logging(options))

def enable_pretty_logging(options=None, logger=None):
    """Turns on formatted logging output as configured.

    This is called automatically by `tornado.options.parse_command_line`
    and `tornado.options.parse_config_file`.
    """
    if options is None:
        import tornado.options
        options = tornado.options.options
    if options.logging is None or options.logging.lower() == 'none':
        return
    if logger is None:
        logger = logging.getLogger()
    logger.setLevel(getattr(logging, options.logging.upper()))
    if options.log_file_prefix:
        rotate_mode = options.log_rotate_mode
        if rotate_mode == 'size':
            channel = logging.handlers.RotatingFileHandler(
                filename=options.log_file_prefix,
                maxBytes=options.log_file_max_size,
                backupCount=options.log_file_num_backups)
        elif rotate_mode == 'time':
            channel = logging.handlers.TimedRotatingFileHandler(
                filename=options.log_file_prefix,
                when=options.log_rotate_when,
                interval=options.log_rotate_interval,
                backupCount=options.log_file_num_backups)
        else:
            error_message = 'The value of log_rotate_mode option should be ' +\
                            '"size" or "time", not "%s".' % rotate_mode
            raise ValueError(error_message)
        channel.setFormatter(LogFormatter(color=False))
        logger.addHandler(channel)

    if (options.log_to_stderr or
            (options.log_to_stderr is None and not logger.handlers)):
        # Set up color if we are in a tty and curses is installed
        channel = logging.StreamHandler()
        channel.setFormatter(LogFormatter())
        logger.addHandler(channel)
```

但是这个函数只是注册了一个回调函数；`enable_pretty_logging`，回调函数被触发调用是在 tornado.options 中通过一个参数被触发的；并且这两个函数是通过外部调用的，也就是说用户决定是否触发；另外 tornado.options 模块中的 options 变量是一个全局变量；可以看到如果 `enable_pretty_logging` 如果被调用，就会通过 `options.log_file_prefix` 参数来设置logging(其实这里是root Logger) handler 为日志文件处理handler。这个就让系统logging可以把日志输出到`log_file_prefix`文件中; 并且此时还会决定是否设置一个控制台输出，这里没有给出 `log_to_stderr` 并且logger 的handler已经不是0了，因为不会设置；那么为什么logging还可以输出到控制台？ 因为在还没使用tornado.options 来设置日志的时候，logging 就被调用了，一旦调用，默认root 首次调用就会调用 basicConfig 来初始化handler，这个handler就是控制台的 logging.StreamHandler; 这里用户使用的 `logging.info('Use config file: %s' % conf_file)` 就是；
```python

options = OptionParser()
"""Global options object.

All defined options are available as attributes on this object.
"""

def parse_command_line(args=None, final=True):
    """Parses global options from the command line.

    See `OptionParser.parse_command_line`.
    """
    return options.parse_command_line(args, final=final)


def parse_config_file(path, final=True):
    """Parses global options from a config file.

    See `OptionParser.parse_config_file`.
    """
    return options.parse_config_file(path, final=final)
def parse_command_line(self, args=None, final=True):
    """Parses all options given on the command line (defaults to
    `sys.argv`).

    Note that ``args[0]`` is ignored since it is the program name
    in `sys.argv`.

    We return a list of all arguments that are not parsed as options.

    If ``final`` is ``False``, parse callbacks will not be run.
    This is useful for applications that wish to combine configurations
    from multiple sources.
    """
    if args is None:
        args = sys.argv
    remaining = []
    for i in range(1, len(args)):
        # All things after the last option are command line arguments
        if not args[i].startswith("-"):
            remaining = args[i:]
            break
        if args[i] == "--":
            remaining = args[i + 1:]
            break
        arg = args[i].lstrip("-")
        name, equals, value = arg.partition("=")
        name = self._normalize_name(name)
        if name not in self._options:
            self.print_help()
            raise Error('Unrecognized command line option: %r' % name)
        option = self._options[name]
        if not equals:
            if option.type == bool:
                value = "true"
            else:
                raise Error('Option %r requires a value' % name)
        option.parse(value)

    if final:
        self.run_parse_callbacks()

    return remaining
def parse_config_file(self, path, final=True):
    """Parses and loads the Python config file at the given path.

    If ``final`` is ``False``, parse callbacks will not be run.
    This is useful for applications that wish to combine configurations
    from multiple sources.

    .. versionchanged:: 4.1
        Config files are now always interpreted as utf-8 instead of
        the system default encoding.

    .. versionchanged:: 4.4
        The special variable ``__file__`` is available inside config
        files, specifying the absolute path to the config file itself.
    """
    config = {'__file__': os.path.abspath(path)}
    with open(path, 'rb') as f:
        exec_in(native_str(f.read()), config, config)
    for name in config:
        normalized = self._normalize_name(name)
        if normalized in self._options:
            self._options[normalized].set(config[name])

    if final:
        self.run_parse_callbacks()

``` 
我们可以看到，用户通过调用 `parse_config_file()` 或者 `parse_command_line()` 传递进来的 final 参数来决定是否配置 tornado 的 `enable_pretty_logging()` 回调函数，来决定日志的配置。下面是一个简单的使用。
```python
#!/usr/bin/env python
def check_options(mandates=None):
    if mandates is None:
        mandates = ['port', 'auth_uri', 'admin_user', 'admin_password',
                'admin_tenant_name']
    for var_name in mandates:
        opt = getattr(options, var_name, None)
        if opt is None or len(str(opt)) == 0:
            raise Exception("No option %s provided" % var_name)


def init(config_path):
    """
    Register all configurable variables
    """
    if config_path is not None:
        config_path = os.path.abspath(config_path)
        conf_name = os.path.basename(config_path)
        search_paths = ['~/xxx', '/usr/local/etc/xxx', '/etc/xxx']
        for p in search_paths:
            tmp_conf = os.path.expanduser(os.path.join(p, conf_name))
            if os.path.exists(tmp_conf):
                config_path = os.path.abspath(tmp_conf)
                logging.info('Use default config %s' % config_path)
                break
    define("config_file",
        default=config_path,
        help="Configuration file")
    define('supervision_mode', type=bool, default=False,
            help='To running services controlled by manager')
    # define('debug', type=bool, default=False,
    #         help='To run service in debug mode')
    define('include_config_file', type=str, default=None,
            help='Extra config file for redefining params')
    define('include_config_dir', type=str, default=None,
            help='Extra config file dir for redefining params')
    define("pid_file", help="Pid file")
    
    project_root_dir = dirname(dirname(dirname(os.path.realpath(__file__))))
    define('locale_dir', type=str, default=(project_root_dir + '/locale/'),
           help='tornado i18n internationalization support')
    setup_locale_translation('cloud', 'zh_CN')

    tornado.options.parse_command_line(final=False)
    conf_file = options.config_file

    logging.info('Use config file: %s' % conf_file)

    if conf_file is None or not os.path.exists(conf_file):
        conf_file = '/dev/null'
    tornado.options.parse_config_file(conf_file, final=False)
    #tornado.options.parse_command_line()

    include_conf_file = options.include_config_file
    if not include_conf_file or not os.path.exists(include_conf_file):
        include_conf_file = '/dev/null'
    tornado.options.parse_config_file(include_conf_file, final=False)

    include_config_dir = options.include_config_dir
    if include_config_dir is not None and os.path.isdir(include_config_dir):
        for each in os.listdir(include_config_dir):
            each_path = os.path.join(include_config_dir, each)
            if os.path.isfile(each_path):
                tornado.options.parse_config_file(each_path, final=False)

    tornado.options.parse_config_file('/dev/null', final=True)

```

# python logging 模块
## 日志处理
Logger: 应用程序日志模块接口, Logger 可以添加 Filter 和 Handler
Filter: 日志记录过滤器，决定哪些日志记录可以发送给 Handler 处理
Handler: 日志处理器，处理 Logger 发送来的 日志记录，决定日志输出到哪里，console、stream、socket等
Formatter: 日志格式器，决定发送给 Handler 的 LogRecord 以什么样式输出

### 日志级别
日志级别，用来设置Logger的日志级别 或者 Handler 的日志处理级别。Logger 自己设置日志级别，可以决定自己到底发不发送该信息给 Handler，Handler设置日志处理级别，决定如果 Logger 发出的 LogRecord 日志级别低于 Handler 自己的日志处理级别， Logger 还是不给 Handler 处理的。
```python
CRITICAL = 50       #logging.critical
FATAL = CRITICAL    #logging.fatal
ERROR = 40          #logging.error
WARNING = 30        #logging.warning
WARN = WARNING      #logging.warn
INFO = 20           #logging.info
DEBUG = 10          #logging.debug
NOTSET = 0          
```
Logger日志级别判断过程的主要代码如下， 只以 info 为例， 其他类似：
```python
def info(self, msg, *args, **kwargs):
    """
    Log 'msg % args' with severity 'INFO'.

    To pass exception information, use the keyword argument exc_info with
    a true value, e.g.

    logger.info("Houston, we have a %s", "interesting problem", exc_info=1)
    """
    if self.isEnabledFor(INFO):
        self._log(INFO, msg, args, **kwargs)

def isEnabledFor(self, level):
    """
    Is this logger enabled for level 'level'?
    """
    if self.manager.disable >= level:
        return 0
    return level >= self.getEffectiveLevel()

def getEffectiveLevel(self):
    """
    Get the effective level for this logger.

    Loop through this logger and its parents in the logger hierarchy,
    looking for a non-zero logging level. Return the first one found.
    """
    logger = self
    while logger:
        if logger.level:
            return logger.level
        logger = logger.parent
    return NOTSET

def _log(self, level, msg, args, exc_info=None, extra=None):
    """
    Low-level logging routine which creates a LogRecord and then calls
    all the handlers of this logger to handle the record.
    """
    if _srcfile:
        #IronPython doesn't track Python frames, so findCaller raises an
        #exception on some versions of IronPython. We trap it here so that
        #IronPython can use logging.
        try:
            fn, lno, func = self.findCaller()
        except ValueError:
            fn, lno, func = "(unknown file)", 0, "(unknown function)"
    else:
        fn, lno, func = "(unknown file)", 0, "(unknown function)"
    if exc_info:
        if not isinstance(exc_info, tuple):
            exc_info = sys.exc_info()
    record = self.makeRecord(self.name, level, fn, lno, msg, args, exc_info, func, extra)
    self.handle(record)

```
我们可以看到每一个 Logger 构造出来的 LogRecord 其实都有个名字，并且这个名字是和 Logger 同名的，也就是说哪个 Logger 输出的日志，就叫什么名字，这个名字是有用处的，除了可以新建不同的 Logger，还可以产生 Logger 的继承关系（并非python代码中的继承），过滤器就是根据自己的名字和 LogRecord 的名字进行过滤操作的;

当然在 Logger 和 Handler 之间可以加过滤器 Filter， 并且过滤器可以加在 Logger 中， 也可以加在 Handler 中。以下是 Logger 的处理代码， 根据加入的 Filter 决定是否把 LogRecord 发给 Handler 处理；
```python
def handle(self, record):
    """
    Call the handlers for the specified record.

    This method is used for unpickled records received from a socket, as
    well as those created locally. Logger-level filtering is applied.
    """
    if (not self.disabled) and self.filter(record):
        self.callHandlers(record)

def callHandlers(self, record):
    """
    Pass a record to all relevant handlers.

    Loop through all handlers for this logger and its parents in the
    logger hierarchy. If no handler was found, output a one-off error
    message to sys.stderr. Stop searching up the hierarchy whenever a
    logger with the "propagate" attribute set to zero is found - that
    will be the last logger whose handlers are called.
    """
    c = self
    found = 0
    while c:
        for hdlr in c.handlers:
            found = found + 1
            if record.levelno >= hdlr.level:
                hdlr.handle(record)
        if not c.propagate:
            c = None    #break out
        else:
            c = c.parent
    if (found == 0) and raiseExceptions and not self.manager.emittedNoHandlerWarning:
        sys.stderr.write("No handlers could be found for logger"
                            " \"%s\"\n" % self.name)
        self.manager.emittedNoHandlerWarning = 1

```
以下是 Handler 自己的处理代码，也就是 Filter 加在 Handler 中；可以看到， Handler 自己也会根据加入到自己的 Filter 进行选择处理；
```python
def handle(self, record):
    """
    Conditionally emit the specified logging record.

    Emission depends on filters which may have been added to the handler.
    Wrap the actual emission of the record with acquisition/release of
    the I/O thread lock. Returns whether the filter passed the record for
    emission.
    """
    rv = self.filter(record)
    if rv:
        self.acquire()
        try:
            self.emit(record)
        finally:
            self.release()
    return rv
```
为什么 Logger 和 Handler 都有 self.filter self.addFilter self.removeFilter 方法，因为 他们都继承了 Filterer 类，该类用来维护和管理 Filter；这个 Filterer 类类似于 Java 中的接口；
```python
class Filterer(object):
    def __init__(self):
        self.filters = []

    def addFilter(self, filter):

    def removeFilter(self, filter):

    def filter(self, record):
        """
        Determine if a record is loggable by consulting all the filters.

        The default is to allow the record to be logged; any filter can veto
        this and the record is then dropped. Returns a zero value if a record
        is to be dropped, else non-zero.
        """
        rv = 1
        for f in self.filters:
            if not f.filter(record):
                rv = 0
                break
        return rv

```
### Logger 继承关系
有时候我们直接使用 logging.xxx 就可以输出日志，这是为什么？因为 logging 模块封装了几个日志处理函数，他们都是这样的, 其实默认都使用了 RootLogger. root 是在 logging 模块导入的时候就初始化了； Python 模块中经常使用这种写法，把一个默认的全局对象实例使用全局函数包装起来方便用户使用；
```python
root = RootLogger(WARNING)
Logger.root = root
Logger.manager = Manager(Logger.root)

def warning(msg, *args, **kwargs):
    """
    Log a message with severity 'WARNING' on the root logger.
    """
    if len(root.handlers) == 0:
        basicConfig()
    root.warning(msg, *args, **kwargs)

def getLogger(name=None):
    """
    Return a logger with the specified name, creating it if necessary.

    If no name is specified, return the root logger.
    """
    if name:
        return Logger.manager.getLogger(name)
    else:
        return root

def basicConfig(**kwargs):
    """
    Do basic configuration for the logging system.

    This function does nothing if the root logger already has handlers
    configured. It is a convenience method intended for use by simple scripts
    to do one-shot configuration of the logging package.

    The default behaviour is to create a StreamHandler which writes to
    sys.stderr, set a formatter using the BASIC_FORMAT format string, and
    add the handler to the root logger.

    A number of optional keyword arguments may be specified, which can alter
    the default behaviour.

    filename  Specifies that a FileHandler be created, using the specified
              filename, rather than a StreamHandler.
    filemode  Specifies the mode to open the file, if filename is specified
              (if filemode is unspecified, it defaults to 'a').
    format    Use the specified format string for the handler.
    datefmt   Use the specified date/time format.
    level     Set the root logger level to the specified level.
    stream    Use the specified stream to initialize the StreamHandler. Note
              that this argument is incompatible with 'filename' - if both
              are present, 'stream' is ignored.

    Note that you could specify a stream created using open(filename, mode)
    rather than passing the filename and mode in. However, it should be
    remembered that StreamHandler does not close its stream (since it may be
    using sys.stdout or sys.stderr), whereas FileHandler closes its stream
    when the handler is closed.
    """
    # Add thread safety in case someone mistakenly calls
    # basicConfig() from multiple threads
    _acquireLock()
    try:
        if len(root.handlers) == 0:
            filename = kwargs.get("filename")
            if filename:
                mode = kwargs.get("filemode", 'a')
                hdlr = FileHandler(filename, mode)
            else:
                stream = kwargs.get("stream")
                hdlr = StreamHandler(stream)
            fs = kwargs.get("format", BASIC_FORMAT)
            dfs = kwargs.get("datefmt", None)
            fmt = Formatter(fs, dfs)
            hdlr.setFormatter(fmt)
            root.addHandler(hdlr)
            level = kwargs.get("level")
            if level is not None:
                root.setLevel(level)
    finally:
        _releaseLock()

```
我们看到 Manager 其实是个日志工厂，使用 logging.getLogger() 方法其实是通过 Manager 获得的，他是通过一个词典维护Logger实例的；上面的代码中， 如果 root.handlers == 0, 还会调用 logging.basicConfig() 方法，这个方法其实是配置 root 的 Handler 用的；默认使用 StreamHandler，StreamHandler 默认会使用控制台输出的；如果想要同时输出到控制台和日志文件，就需要自己手动添加 Handler 给 RootLogger 或者自己新建的 Logger；
```python
import logging
logging.basicConfig(
                   filename = './log.txt',
                   filemode = 'a',
                   #stream = sys.stdout,
                   format = '%(levelname)s:%(message)s',
                   datefmt = '%m/%d/%Y %I:%M:%S',
                   level = logging.DEBUG
                    )

logging.debug('10')
logging.info('20')
logging.warn('30')
logging.warning('30')
logging.error('40')
logging.exception('40')
logging.fatal('50')
logging.critical('50')
```
除了 RootLogger， 其他有名字的Logger是如何构造出来的？他们的继承关系是怎么样的？其实是通过简单的名字来判断继承关系。在获取 Logger 的时候，如果这个Logger 还不存在即没有记录在词典中，那么就直接生成新Logger，并且跟踪父子关系。如果已经存在，首先判断是不是一个PlaceHolder，就是还没有这个 Logger，但是为了维持父子关系，需要一个占位节点，然后跟踪记录父子关系。
```python
def getLogger(self, name):
    """
    Get a logger with the specified name (channel name), creating it
    if it doesn't yet exist. This name is a dot-separated hierarchical
    name, such as "a", "a.b", "a.b.c" or similar.

    If a PlaceHolder existed for the specified name [i.e. the logger
    didn't exist but a child of it did], replace it with the created
    logger and fix up the parent/child references which pointed to the
    placeholder to now point to the logger.
    """
    rv = None
    if not isinstance(name, basestring):
        raise TypeError('A logger name must be string or Unicode')
    if isinstance(name, unicode):
        name = name.encode('utf-8')
    _acquireLock()
    try:
        if name in self.loggerDict:
            rv = self.loggerDict[name]
            if isinstance(rv, PlaceHolder):
                ph = rv
                rv = (self.loggerClass or _loggerClass)(name)
                rv.manager = self
                self.loggerDict[name] = rv
                self._fixupChildren(ph, rv)
                self._fixupParents(rv)
        else:
            rv = (self.loggerClass or _loggerClass)(name)
            rv.manager = self
            self.loggerDict[name] = rv
            self._fixupParents(rv)
    finally:
        _releaseLock()
    return rv

def _fixupParents(self, alogger):
    """
    Ensure that there are either loggers or placeholders all the way
    from the specified logger to the root of the logger hierarchy.
    """
    name = alogger.name
    i = name.rfind(".")
    rv = None
    while (i > 0) and not rv:
        substr = name[:i]
        if substr not in self.loggerDict:
            self.loggerDict[substr] = PlaceHolder(alogger)
        else:
            obj = self.loggerDict[substr]
            if isinstance(obj, Logger):
                rv = obj
            else:
                assert isinstance(obj, PlaceHolder)
                obj.append(alogger)
        i = name.rfind(".", 0, i - 1)
    if not rv:
        rv = self.root
    alogger.parent = rv
```
最后， Logger 还有一个 propagate 属性，决定callHandler的时候，是否将LogRecord 发送给 父亲的Handler继续处理。