---
title: "flask"
date: 2017-04-23 21:23
---

# flaskr 开发笔记

## Python 虚拟环境使用
虚拟环境很好使，推荐使用，他不会影响你的Ubuntu自带的Python环境，相当于重新创建了一个Python环境，然后在这个环境下进行包的安装和开发！以下是简单的安装和使用方法

```
// 可能没得virtaualenv，先安装
sudo apt-get install python-pip python-dev python-virtualenv
// 然后新建一个目录,比如你的项目
mkdir todo-api
cd todo-api
// 该句会创建一个flask目录，就是一个新的Python虚拟环境，里面的目录结构其实和主机上Python目录结构很类似，都有bin目录下的命令，比如pip，python，activate等
virtualenv flask
//然后你可以使用flask/bin/pip安装项目需要的依赖包，此时安装的就是Flask包了，他在flask/lib/python2.7/site-packages目录下面，不会影响local Python的。
flask/bin/pip install flask
//如果你想要运行.py文件，可以直接使用
flask/bin/python app.py
//当然还可以使用
chmod a+x app.py
./app.py

```
## web app 配置方式
>Flask allows you to import multiple configurations and it will use the setting defined in the last import. 
从文件导入配置

```python
app.config.from_pyfile(filename, silent=False)
```
从对象导入配置

```python
app.config.from_object(obj)
	obj: string, 叫这个名字的模块会被import,也可以是一个直接已经导入的object
```
从环境变量指定的地方导入

```python
app.config.from_envvar(variable_name, silent=False)
```
从一个词典导入，运行时更新

```python
app.config.update(dict)
```

## flask 引入上下文的概念
为了保证同一个请求共享数据库连接，而不是反复connect_db(),flask引入applocation context的概念，g 是全局共享的

```python
def get_db():
	"""Opens a new database connection if there is none yet for the
    current application context.
    """
	if not hasattr(g, 'sqlite_db'):
		g.sqlite_db = connect_db()
	return g.sqlite_db
```

## flask 提供命令行接口，并可以读取APP资源
```
def init_db():
	db = get_db()
	with app.open_resource('schema.sql', mode='r') as f:
		db.cursor().executescript(f.read())
	db.commit()
# flask的命令行接口创建数据库
@app.cli.command('initdb')
def initdb_command():
	init_db()
	print 'Initialized the database.'
```