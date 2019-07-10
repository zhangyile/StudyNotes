# Python

## Topic

### pretty print  

```shell
# shell 命令行输出
$ echo '{"json":"obj"}' | python -m json.tool
{
    "json": "obj"
}
$ python -m json.tool < test.json
``` 
```python
# python 第三方模块
import pprint
pprint.pprint(dest_dict)
```

### Create Random String

#### UUID
Universally Unique IDentifier 

1、`uuid1()`——基于时间戳
由MAC地址、当前时间戳、随机数生成。可以保证全球范围内的唯一性，
但MAC的使用同时带来安全性问题，局域网中可以使用IP来代替MAC。

2、`uuid2()`——基于分布式计算环境DCE（Python中没有这个函数）
算法与uuid1相同，不同的是把时间戳的前4位置换为POSIX的UID。
实际中很少用到该方法。

3、`uuid3()`——基于名字的MD5散列值
通过计算名字和命名空间的MD5散列值得到，保证了同一命名空间中不同名字的唯一性，
和不同命名空间的唯一性，但同一命名空间的同一名字生成相同的uuid。    

4、`uuid4()`——基于随机数
由伪随机数得到，有一定的重复概率，该概率可以计算出来。

5、`uuid5()`——基于名字的SHA-1散列值
算法与uuid3相同，不同的是使用 Secure Hash Algorithm 1 算法

使用方面：
首先，Python中没有基于DCE的，所以uuid2可以忽略；
其次，uuid4存在概率性重复，由无映射性，最好不用；
再次，若在Global的分布式计算环境下，最好用uuid1；
最后，若有名字的唯一性要求，最好用uuid3或uuid5。

```python
# -*- coding: utf-8 -*-

import uuid

name = "test_name"
namespace = "test_namespace"

print uuid.uuid1()  # 带参的方法参见Python Doc
print uuid.uuid3(namespace, name)
print uuid.uuid4()
print uuid.uuid5(namespace, name)

```

    
### docorator

作用：（赋予装饰对象一些通用属性）
简单的例子解释装饰器的作用
[参考文档](https://www.cnblogs.com/Jerry-Chou/archive/2012/05/23/python-decorator-explain.html)

```python
// 生成 p 标签

def pDocorator(label):
    def decorator(func):
        def _decorator(text):
            return '<{0}>{1}</{0}>'.format(label, func(text))
        return _decorator
    return decorator

@pDocorator('p')
def getString(text):
    return text

getString('zyl')
>>> '<p>zyl</p>'

```

#### @property

作用： 对实例属性操作时进行包装

```python
class Student(object):

    # 读取时触发
    @property
    def birth(self):
        return self._birth
    # 赋值时触发
    @birth.setter 
    def birth(self, value): 
        self._birth = value

    # 只读属性
    @property  
    def age(self):
        return 2014 - self._birth
```

#### @staticmethod & @classmethod

作用： 不需要实例化，都可以直接通过类名.方法名()来调用
@staticmethod 不需要表示自身对象的self和自身类的cls参数，就跟使用函数一样
@classmethod 不需要self参数，但第一个参数需要是表示自身类的cls参数。

如果在@staticmethod中要调用到这个类的一些属性方法，只能直接类名.属性名或类名.方法名。
而@classmethod因为持有cls参数，可以来调用类的属性，类的方法，实例化对象等，避免硬编码。


```python
class A(object):  
    bar = 1  
    def foo(self):  
        print 'foo'  
 
    @staticmethod  
    def static_foo():  
        print 'static_foo'  
        print A.bar  
 
    @classmethod  
    def class_foo(cls):  
        print 'class_foo'  
        print cls.bar  
        cls().foo()  
  
A.static_foo()  
A.class_foo()  
```



#### @abstractmethod

作用： 被该装饰器修饰的方法，子类都必须实现这个方法，不然无法实例化对象。



### encode & decode

> *字符串在Python内部的表示是Unicode编码*

* encode 编码 (编译为其他编码)
 Unicode  -> 'gbk/utf-8/etc'  
     
* decode 解码 (转为Unicode)
'gbk/utf-8/gb2312' -> Unicode

    ```python
    print sys.getdefaultencoding()  #获取系统默认的编码
    reload(sys)
    sys.setdefaultencoding('utf8')  #修改系统的默认编码
    print sys.getdefaultencoding()
    
    In [86]: a = u'张毅乐'

    In [87]: a
    Out[87]: u'\u5f20\u6bc5\u4e50'
    
    In [88]: a.encode('gbk')
    Out[88]: '\xd5\xc5\xd2\xe3\xc0\xd6'
    
    In [94]: a.encode('utf-8')
    Out[94]: '\xe5\xbc\xa0\xe6\xaf\x85\xe4\xb9\x90'
    ```

### command parser


#### optparse

example：

```python
from optparse import OptionParser
...
parser = OptionParser()
parser.add_option("-f", "--file", dest="filename",
                  help="write report to FILE", metavar="FILE")
parser.add_option("-q", "--quiet",
                  action="store_false", dest="verbose", default=True,
                  help="don't print status messages to stdout")

(options, args) = parser.parse_args()

# python ./XXX.py -f zyl -q test

option  # {'verbose': False, 'filename': 'zyl'}
args    # ['test']
```

#### parser

### 命名规范

![](media/15163481845232/15457096516796.jpg)


## module

### built in function  

#### super

用法： 调用自身(`self.__class__`)的父类(`self.___bases__`)的属性或者方法 
示例如下：    
 
```python 
class A(object):
    def __init__(self):
        print 'A'
class B(A):
    def __init__(self):
        super(B, self).__init__()
        print 'B'
       
class C(B, A):  # it's ok
    pass
  
class D(A, B):  # Raise Error
    pass
```

#### assert 

解释：当表达式为 `True` 时，pass
为`False`时 ， raise AssertionError

```python
assert expression [, error_info] 
    
>>> assert 1 < 0, 'this is error info'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError: this is error info
>>>
```

#### datetime & time

|格式符|说明|
| --- | --- |
|%a|星期的英文单词的缩写：如星期一， 则返回 Mon|
|%A|	星期的英文单词的全拼：如星期一，返回 Monday|
|%b|	月份的英文单词的缩写：如一月， 则返回 Jan|
|%B|	月份的引文单词的缩写：如一月， 则返回 January|
|%c|	返回datetime的字符串表示，如03/08/15 23:01:26|
|%d|	返回的是当前时间是当前月的第几天|
|%f|	微秒的表示： 范围: [0,999999]|
|%H|	以24小时制表示当前小时|
|%I|	以12小时制表示当前小时|
|%j|	返回 当天是当年的第几天 范围[001,366]|
|%m|	返回月份 范围[0,12]|
|%M|	返回分钟数 范围 [0,59]|
|%P|	返回是上午还是下午–AM or PM|
|%S|	返回秒数 范围 [0,61]。。。手册说明的|
|%U|	返回当周是当年的第几周 以周日为第一天|
|%W|	返回当周是当年的第几周 以周一为第一天|
|%w|	当天在当周的天数，范围为[0, 6]，6表示星期天|
|%x|	日期的字符串表示 ：03/08/15|
|%X|	时间的字符串表示 ：23:22:08|
|%y|	两个数字表示的年份 15|
|%Y|	四个数字表示的年份 2015|
|%z|	与utc时间的间隔 （如果是本地时间，返回空字符串）|
|%Z|	时区名称（如果是本地时间，返回空字符串）|

```python
>>> import datetime
>>> from pytz import tiemzone
>>> now = datetime.datetime.now()
datetime.datetime(2018, 4, 23, 21, 53, 23, 356373)
>>> now.replace(tzinfo=timezone("Asia/Shanghai"))
datetime.datetime(2018, 4, 23, 21, 53, 23, 356373, tzinfo=<DstTzInfo 'Asia/Shanghai' LMT+8:06:00 STD>)
    
# converting datetime to string
>>> datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
'2017-11-21 23:24:41'
    
# converting string to datetime
>>> datetime.datetime.strptime("2017-11-21", "%Y-%m-%d")
datetime.datetime(2017, 11, 21, 0, 0)
>>> datetime.datetime.strptime("2017-11-21T21:42:23.123456Z", "%Y-%m-%dT%H:%M:%S.%fZ")
datetime.datetime(2017, 11, 21, 21, 42, 23, 123456)
    
# converting datetime to timestamp
>>> import time, datetime
>>> time.time()
1511279385.93834
>>> time.mktime(datetime.datetime.now().timetuple())
1511279385.0    #float
>>> datetime_instance.strftime('%s')
'1511271743'   # string

# converting timestamp to datetime
>>> datetime.datetime.fromtimestamp(float('1511271743'))
datetime.datetime(2017, 11, 21, 21, 42, 23)

```

#### map filter reduce

**map**

```python
def add(x):
    return x**2

array = [1, 2, 3, 4]
    
result = map(add, array)
list(result)
>>> [1, 4, 9, 16]
```

**filter**

```python
result = filter(lambda x: x < 3, array)
list(result)

>>> [1, 2]
```

**reduce**

```python
from functools import reduce
reduce(lambda x, y: x + y, array)

>>> 10
```

#### file

1. use `with` R/W file  

 ```python
with open(r'somepath/file.txt') as f:
    for line in f:
        print line
```
2. use `open`  

    ```python
    f = open(r'somepath/file.txt', 'r')
    while True:
        line = f.readline()
        if not line: break
        print line  
    f.close()
    ```

#### socket

创建一个TCP/IP socket
`tcpsocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)`
创建一个UDP/IP socket
`udpsocket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)`


#### Threading & multiprocessing


```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
import threading
import time
 
class myThread (threading.Thread):
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
    def run(self):
        print "Starting " + self.name
       # 获得锁，成功获得锁定后返回True
       # 可选的timeout参数不填时将一直阻塞直到获得锁定
       # 否则超时后将返回False
        threadLock.acquire()
        print_time(self.name, self.counter, 3)
        # 释放锁
        threadLock.release()
 
 
def print_time(threadName, delay, counter):
    while counter:
        time.sleep(delay)
        print "%s: %s" % (threadName, time.ctime(time.time()))
        counter -= 1
 
threadLock = threading.Lock()
threads = []
 
# 创建新线程
thread1 = myThread(1, "Thread-1", 1)
thread2 = myThread(2, "Thread-2", 2)
 
# 开启新线程
thread1.start()
thread2.start()
 
# 添加线程到线程列表
threads.append(thread1)
threads.append(thread2)
 
# 等待所有线程完成
for t in threads:
    t.join()
print "Exiting Main Thread"
```




```python
import threading

class MyThread(threading.Thread):
    def __init__(self, func, args, name=""):
        threading.Thread.__init__(self)
        self.name = name
        self.func = func
        self.args = args

    def getResult(self):
        return self.res

    def run(self):
        self.res = apply(self.func, self.args)
        
def print_time(delay, counter):
    while counter:
        time.sleep(delay)
        counter -= 1
        print delay, counter

ThreadList = []
for i in range(5):
    ThreadList.append(MyThread(print_time, (i, i))
    
for item in ThreadList:
    item.start()
    
for item in ThreadList:
    item.join()
    res = item.getResult()
    print res

```

-------


### 常用模块


#### re

[正则表达式速查表](https://www.jb51.net/shouce/jquery1.82/regexp.html)

| 字符 | 描述  |
| --- | --- |
| ^ | 匹配输入字符串的开始位置。如果设置了RegExp对象的Multiline属性，^也匹配“\n”或“\r”之后的位置。|
| $ | 匹配输入字符串的结束位置。如果设置了RegExp对象的Multiline属性，也匹配“\n”或“\r”之前的位置 |
| . | 匹配除“\n”之外的任何单个字符。要匹配包括“\n”在内的任何字符，请使用像“(.|\n)”的模式。|
| * | 匹配前面的子表达式零次或多次。例如，zo*能匹配“z”以及“zoo”。*等价于{0,}。 |
| + | 匹配前面的子表达式一次或多次。例如，“zo+”能匹配“zo”以及“zoo”，但不能匹配“z”。+等价于{1,}。 |
| ? | 匹配前面的子表达式零次或一次。例如，“do(es)?”可以匹配“does”或“does”中的“do”。?等价于{0,1}|
| \d | 匹配一个数字字符。等价于[0-9] |


**match**
匹配string 开头，成功返回Match object, 失败返回None，只匹配一个。

```python
import re
string = 'abc123def456'
Obj = re.compile(r'.*?(\d+).*?')

res = Obj.match(string)
res.groups()

# ('123',)
```

**search**
在string中进行搜索，成功返回Match object, 失败返回None, 只匹配一个。

```python
import re
content = '333STR1666STR299'
regex = r'[A-Z]+(\d)'

match = re.search(regex, content)
print('\nre.search() return value: ' + str(type(match)))
print(match.group(0), match.group(1))  

# re.search() return value: <TYPE ?_sre.SRE_Match?>
# STR1 1 
```

**findall**
在string中查找所有匹配成功的组, 即用括号括起来的部分。返回list对象，每个list item是由每个匹配的所有组组成的list。

```python
import re
string = 'abc123def456'
Obj = re.compile(r'\d+')

res = Obj.findall(string)
print res

# ('123', '456')
```

**finditer**
在string中查找所有匹配成功的字符串, 返回iterator，每个item是一个Match object。


#### psutil


```python
import psutil

```

#### redis

**Base Usage**

```python
import redis
r = redis.StrictRedis(host='localhost', port=1234, db=0)
r.set('foo', 'bar')  # True
r.get('foo')  # 'bar'
```


**Sentinel Support**

```python
import redis
from redis.sentinel import Sentinel

sentinel = Sentinel([('localhost', 10110)], socket_timeout=0.1)
sentinel.discover_master('master_name')  # ('127.0.0.1', 6379)
sentinel.discover_slave('master_name')  # ('127.0.0.1', 6380)

master = sentinel.master_for('mymaster', socket_timeout=0.1)
slave = sentinel.slave_for('mymaster', socket_timeout=0.1)
master.set('foo', 'bar')
slave.get('foo')  # 'bar'
```

#### MySQLdb

1. base usage
 
    ```python
    import MySQLdb
    
    conn = MySQLdb.connect(host='', port=, user='', passwd='')
    cursor = conn.cursor()
    sql = 'select * from db.tb'
    change_id = cursor.execute(sql)
    
    one_raw = cursor.fetchone()  #获取一条记录
    all_raw = cursor.fetchall()  #获取所有记录
    
    cursor.close()
    conn.commit()
    conn.close()
    
    ```
    
#### influxdb
    
`pip install influxdb`

```python
import influxdb
influxdb_conf = ('1.1.1.1', 8086, 'xxx', 'xxxxxxxx')
client = influxdb.InfluxDBClient(*influxdb_conf)

qs = 'select last("maxmemory") as "maxmemory", "used_memory" from "redis_node_metrics".."redis" where host=\'{}\' and port=\'{}\''

r = client.query(qs.format('1.1.1.2', 12345), epoch="s") # epoch 指定输出的time为时间戳
if r:
    generator = r.get_points("redis")
    print generator.next()
    
>>> {u'maxmemory': 1073741824, u'time': 1524542003, u'used_memory': 961296}

```
    
#### pymongo
 
 > [DRF integration tutorial](https://medium.com/@vasjaforutube/django-mongodb-django-rest-framework-mongoengine-ee4eb5857b9a)
 
```python
import pymongo
client = pymongo.MongoClient($uri, $port)

d = datetime.datetime.now()
db = client.db_name
db.gather.find({"date": {"$gt": d}}).sort("date", 1)

```
 
 
 
#### ConfigParser

Load config

```python
# ./config.ini 
[host]
name: localhost1
```  

```python
import ConfigParser
configfile_name = "config.ini"

with open(configfile_name) as f:
    cfgfile = f.read()
    
config = ConfigParser.RawConfigParser(allow_no_value=True)
config.readfp(io.BytesIO(cfgfile))
```

Read config

```python
for section in config.sections():
    print section
    for options in config.options(section):
        print options, config.get(section, options)

# output
host
name, localhost

```

Write config

```python

Config = ConfigParser.ConfigParser()
Config.add_section('host')
Config.set('host', 'name', 'localhost')

cfgfile = open(r'./config.ini', w)
Config.write(cfgfile)
cfgfile.close()
```

#### requests & urllib2

1. urllib2  

    ```python
    # url 编码
    In [19]: url = '张毅乐'
    
    In [20]: urllib2.quote(url)
    Out[20]: '%E5%BC%A0%E6%AF%85%E4%B9%90'
    ```
        
    ```python
    import urllib2
    url = 'https://www.163.com'
    
    
    result = urllib2.urlopen(url)
    
    result.code         # 请求返回的状态码
    result.header       # 请求返回的Header
    result.read         # 以文件句柄的方式操作
    result.readline     # 读取一行
    result.readlines    # 读取所有内容，返回以行为单位的列表
    ```

2. requests  

 ```python
>>> import requests
>>> r = requests.get('https://api.github.com/user', auth=('user', 'pass'), params={})
>>> r.status_code
200
>>> r.headers['content-type']
'application/json; charset=utf8'
>>> r.encoding
'utf-8'
>>> r.text
u'{"type":"User"...'
>>> r.json()
{u'private_gists': 419, u'total_private_repos': 77, ...}
```

#### pika

RabbitMQ 简单使用示例：

**Producer**

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='hello')
channel.basic_publish(exchange='', 
                      routing_key='hello', 
                      body='Hello World')
connection.close()
print("Sent Message")
```

**Consumer**

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
channel.queue_declare(queue='hello')

def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)

channel.basic_consume(callback,
                      queue='hello',
                      no_ack=True)
print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```


## Basic

### yield

> [参考资料](https://stackoverflow.com/questions/231767/what-does-the-yield-keyword-do/231855#231855)

```python
>>> def createGenerator():
...    mylist = range(3)
...    for i in mylist:
...        yield i*i
...
>>> mygenerator = createGenerator() # create a generator
>>> print(mygenerator) # mygenerator is an object!
<generator object createGenerator at 0xb7555c34>
>>> for i in mygenerator:
...     print(i)
0
1
4
```

### Generators

首先让我们了解迭代器。根据Wikipedia的说法，迭代器是一个对象，它允许程序员遍历容器，特别是列表。然而，迭代器执行遍历并访问容器中的数据元素，但不执行迭代。

**Understanding the inner mechanisms of iteration**

Iteration is a process implying iterables (implementing the __iter__() method) and iterators (implementing the __next__() method). Iterables are any objects you can get an iterator from. Iterators are objects that let you iterate on iterables.


### *args & **kwargs

* Usage of *args and **kwargs

```python
def test_var_args(f_arg, *argv):
    print("first normal arg:", f_arg)
    for arg in argv:
        print("another arg through *argv:", arg)

test_var_args('yasoob', 'python', 'eggs', 'test')

# first normal arg: yasoob
# another arg through *argv: python
# another arg through *argv: eggs
# another arg through *argv: test


def greet_me(**kwargs):
    for key, value in kwargs.items():
        print("{0} = {1}".format(key, value))

greet_me(name="yasoob")
# name = yasoob
```

* Using *args and **kwargs to call a function

```python
def test_args_kwargs(arg1, arg2, arg3):
    print("arg1:", arg1)
    print("arg2:", arg2)
    print("arg3:", arg3)
    
# first with *args
>>> args = ("two", 3, 5)
>>> test_args_kwargs(*args)
arg1: two
arg2: 3
arg3: 5

# now with **kwargs:
>>> kwargs = {"arg3": 3, "arg2": "two", "arg1": 5}
>>> test_args_kwargs(**kwargs)
arg1: 5
arg2: two
arg3: 3

```

### RSA



----------------------

# django 

## Topic

### 多数据库配置

（django multiple database configuration）

1. 首先在 setting 中配置不同的数据库  
 
    ```python
    # setting.py
    
    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
    
        'pedis',    # INSTALLED_APPS 中需要有该应用，否则 migrate 时不会自动创建表
    ]
    
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        },
        'pedis': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME':     'pedis_zyl',
            'USER':     'pedis',
            'PASSWORD': 'XXXXX',
            'HOST':     'XXXXX',
            'PORT':     '3306',
        },
    }
    ```

2. 编写 pedis 应用中的 models：  

    ```python
    # pedis/models.py
    
    # -*- coding: utf-8 -*-
    from __future__ import unicode_literals
    
    from django.db import models
    
    # Create your models here.
    
    class CommonInfo(models.Model):
        name = models.CharField(max_length=100)
        age = models.CharField(max_length=100)
    
        class Meta:
            db_table  = 'hostpool'  # 声明创建数据库时的表名
            app_label = 'pedis'     # 指定该表的数据都从 pedis 中读取，
                                    # 该 pedis 对应的是 setting.DATABASES.pedis
    
        def __str__(self):
            return self.name
    ```

3. 编辑数据库的路由 database router  

    ```python
    # database_router.py
    
    class CommonInfoRouter(object):
        """
        A router to control all database operations on models in the
        auth application.
        
        
        """
        def db_for_read(self, model, **hints):
            """
            Attempts to read auth models go to auth_db.
            return None # 表示没有建议， 
            return ’db_name’ # 表示该 app_label 的数据从 db_name 数据库中读取
            检索完所有的路由表后建议为 None，则默认从 default 数据库中读取
            """
            if model._meta.app_label == 'auth':
                return 'auth'
            return None
    
        def db_for_write(self, model, **hints):
            """
            Attempts to write auth models go to auth_db.
            """
            if model._meta.app_label == 'auth':
                return 'auth'
            return None
    
        def allow_relation(self, obj1, obj2, **hints):
            """
            Allow relations if a model in the auth app is involved.
            """
            if obj1._meta.app_label == 'auth' or \
               obj2._meta.app_label == 'auth':
               return True
            return None
    
        def allow_migrate(self, db, app_label, model_name=None, **hints):
            """
            Make sure the auth app only appears in the 'auth_db'
            database.
            """
            if app_label == 'auth':
                return db == app_label
            elif db == 'auth_db':
                return False
            return None
    ```

4. 配置数据库路由：

    ```python
    # setting.py
    
    DATABASE_ROUTERS = ['database_router.CommonInfoRouter']
    ```

5. 同步数据库，（即建表建库）

    `python manage.py makemigrations pedis` # 生成建表语句  
    `python manage.py migrate pedis --database=pedis` # 建表 


### CSRF 

Cross-Site request forgery 跨站请求伪造

基于CSRF的一次成功Post请求流程
1. 用户登录成功后，系统会生成一个随机字符串 放在请求头的 csrf-token 字段
2. 当用户发起post请求时，必须携带 csrf-token
3. Server端会判断该请求携带 csrf-token 是否正确，正确则接受该请求，否则提示token无效，拒绝该请求 


### Model

#### 常用操作

##### 字段值自增

```python
from django.contrib.auth.models import User
from django.db.models import F

user = User.objects.filter(id__in=[1,2,3])
user.update(age=F('age') + 10)
```

##### Retrieval values of special field

```python
user = User.objects.values_list(‘id’, flat=True).all()
print user
# [1,2,3]

```    

##### complex query Q

```python
from django.db.models import Q
condition = Q(name__in=['zyl', 'sr'])
condition != Q(name='sunrui')
condition &= Q(name='zhangyile')
```

##### querying ArrayField

`contains`
The returned objects will be those where the values passed are a subset of the data

`contained_by`
the objects returned will be those where the data is a subset of the values passed


# Django Rest Framework 

## 基本概念

### 专业术语解释

| 简称 | 全称 | 解释 |
| --- | --- | --- |
| MVC | Model, View, Control |
| WSGI | Web Server Gateway Interface | Web服务器和Web应用程序之间通用的网关a接口 |



## 框架架构总览及核心流程图

架构总览
![](media/15163481845232/15265264154362.jpg)



中间件 Middleware 都需要在 “project/settings.py” 中 MIDDLEWARE_CLASSES 的定义。一个 HTTP 请求，将被这里指定的中间件从头到尾处理一遍，暂且称这些需要挨个处理的中间件为处理链，
**如果链中某个处理器处理后没有返回 response，就把请求传递给下一个处理器；如果链中某个处理器返回了 response，直接跳出处理链由 response 中间件处理后返回给客户端，可以称之为短路处理。**
![](media/15163481845232/15265266043677.jpg)




# Ansible 运行原理

## `ansible-playbook test.yml`

输出是这样子的：

```bash

PLAY [10.160.246.204] *************************************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************************************************
ok: [10.160.246.204]

TASK [tomcat_api_zyl_test : list /home/] ******************************************************************************************************************************************
changed: [10.160.246.204]

TASK [tomcat_api_zyl_test : list /home/ {{ customer }}] ***************************************************************************************************************************
fatal: [10.160.246.204]: FAILED! => {"changed": true, "cmd": ["ls", "-l", "/home/hzzhangyile"], "delta": "0:00:00.061202", "end": "2018-02-24 15:06:15.116735", "msg": "non-zero return code", "rc": 2, "start": "2018-02-24 15:06:15.055533", "stderr": "ls: 无法访问/home/hzzhangyile: 没有那个文件或目录", "stderr_lines": ["ls: 无法访问/home/hzzhangyile: 没有那个文件或目录"], "stdout": "", "stdout_lines": []}
	to retry, use: --limit @/home/webedit/hzzhangyile/python_script/ansible_zyl/zyl_test.retry

PLAY RECAP ************************************************************************************************************************************************************************
10.160.246.204             : ok=2    changed=1    unreachable=0    failed=1

Gathering Facts --------------------------------------------------------- 3.31s
list /home/ ------------------------------------------------------------- 0.74s
list /home/ {{ customer }} ---------------------------------------------- 0.53s

Playbook finished: Sat Feb 24 15:06:15 2018, 3 total tasks.  0:00:04 elapsed.

Gathering Facts --------------------------------------------------------- 3.31s
list /home/ ------------------------------------------------------------- 0.74s
list /home/ {{ customer }} ---------------------------------------------- 0.53s

Playbook finished: Sat Feb 24 15:06:15 2018, 3 total tasks.  0:00:04 elapsed.

Playbook run took 0 days, 0 hours, 0 minutes, 4 seconds
```

这条命令做了哪些事呢？先来看一下test.yml 是如何写的。

> cat test.yml

```
---

- hosts: 10.160.246.204

  tasks:
    - name: list host directory
      command: ls -l ~/

  roles:
   - role: tomcat_api_zyl_test
```

源码里是这样的  

```python
# 动态导入 ansible.cli.playbook
# 相当于 from ansible.cli.playbook import PlaybookCLI as mycli
mycli = getattr(__import__("ansible.cli.%s" % sub, fromlist=[myclass]), myclass)
```

主要步骤在这：


```python
try:
    args = [to_text(a, errors='surrogate_or_strict') for a in sys.argv]
except UnicodeError:
    display.error('Command line args are not in utf-8, unable to continue.  Ansible currently only understands utf-8')
    display.display(u"The full traceback was:\n\n%s" % to_text(traceback.format_exc()))
    exit_code = 6
else:
    cli = mycli(args)       # 1. 本次运行命令的实例化
    cli.parse()             # 2. 解析命令参数
    exit_code = cli.run()   # 3. 运行命令
```

接下来，我们详细的看一下，这三步都干什么了

##### 1. 运行命令的实例化

`cli = mycli(args)` 就是实例化`ansible.cli.playbook.PlaybookCLI` 这个类，而这个类是这样定义的： 

```python
from ansible.cli import CLI
...

class PlaybookCLI(CLI):
    ''' balabala '''
    def parse(self):
```
可以看出`PlaybookCLI`这个类只是重写了一些方法，并没有修改`__init__()`这个方法，所以，来看看 CLI 中的 `__init__()` 都干嘛了
 
 
```python
class CLI(with_metaclass(ABCMeta, object)):
    ''' code behind bin/ansible* programs '''

    VALID_ACTIONS = []

    _ITALIC = re.compile(r"I\(([^)]+)\)")
    _BOLD = re.compile(r"B\(([^)]+)\)")
    _MODULE = re.compile(r"M\(([^)]+)\)")
    _URL = re.compile(r"U\(([^)]+)\)")
    _CONST = re.compile(r"C\(([^)]+)\)")

    PAGER = 'less'

    # -F (quit-if-one-screen) -R (allow raw ansi control chars)
    # -S (chop long lines) -X (disable termcap init and de-init)
    LESS_OPTS = 'FRSX'
    SKIP_INVENTORY_DEFAULTS = False

    def __init__(self, args, callback=None):
        """
        Base init method for all command line programs
        """

        self.args = args
        self.options = None
        self.parser = None
        self.action = None
        self.callback = callback

```

重点就是 `CLI.__init__()` 这个方法，这个方法的作用就是初始化一些变量，并且将命令行的参数给了 `self.args`， 直观点就是这样子`self.args = ['test.yml']`

到此，实例化对象完成了，还有其他的各种异常判断、功能扩展什么的，就先不深入研究了
    
    
    
##### 2. 解析命令参数

`cli.parse()` 这个方法是这样写的（看里面的注释行）：

```python
   def parse(self):

        # create parser for CLI options
        # 所有的初始化配置、基本上全是在CLI.base_parser()里实现的。
        # 有兴趣的可以详细看一下这个 方法的实现，后期有空，我再针对该方法进行详细解读。
        parser = CLI.base_parser(
            usage="%prog [options] playbook.yml [playbook2 ...]",
            connect_opts=True,
            meta_opts=True,
            runas_opts=True,
            subset_opts=True,
            check_opts=True,
            inventory_opts=True,
            runtask_opts=True,
            vault_opts=True,
            fork_opts=True,
            module_opts=True,
            desc="Runs Ansible playbooks, executing the defined tasks on the targeted hosts.",
        )

        # ansible playbook specific opts
        parser.add_option('--list-tasks', dest='listtasks', action='store_true',
                          help="list all tasks that would be executed")
        parser.add_option('--list-tags', dest='listtags', action='store_true',
                          help="list all available tags")
        parser.add_option('--step', dest='step', action='store_true',
                          help="one-step-at-a-time: confirm each task before running")
        parser.add_option('--start-at-task', dest='start_at_task',
                          help="start the playbook at the task matching this name")

        self.parser = parser    # 实例化 parser 对象，用来解析命令行参数
        super(PlaybookCLI, self).parse() # 解析参数
        # 上面这行代码其实就等于 self.options, self.args = self.parser.parse_args(self.args[1:])
        if len(self.args) == 0:
            raise AnsibleOptionsError("You must specify a playbook file to run")

        display.verbosity = self.options.verbosity
        self.validate_conflicts(runas_opts=True, vault_opts=True, fork_opts=True)
```

这个方法做的工作就是初始化一些系统预设的变量，然后把 `self.options, self.args = self.parser.parse_args(self.args[1:])` 
你可以直观的理解为 self.args = 'test.yml'，

##### 3. 运行命令

`exit_code = cli.run()` 这个方法就是实际运行命令了。先来看源码的实现

```python
    def run(self):

        super(PlaybookCLI, self).run()

        # Note: slightly wrong, this is written so that implicit localhost
        # Manage passwords
        sshpass = None
        becomepass = None
        passwords = {}

        # initial error check, to make sure all specified playbooks are accessible
        # before we start running anything through the playbook executor
        for playbook in self.args:
            if not os.path.exists(playbook):
                raise AnsibleError("the playbook: %s could not be found" % playbook)
            if not (os.path.isfile(playbook) or stat.S_ISFIFO(os.stat(playbook).st_mode)):
                raise AnsibleError("the playbook: %s does not appear to be a file" % playbook)

        # don't deal with privilege escalation or passwords when we don't need to
        if not self.options.listhosts and not self.options.listtasks and not self.options.listtags and not self.options.syntax:
            self.normalize_become_options()
            (sshpass, becomepass) = self.ask_passwords()
            passwords = {'conn_pass': sshpass, 'become_pass': becomepass}

        loader, inventory, variable_manager = self._play_prereqs(self.options)

        # (which is not returned in list_hosts()) is taken into account for
        # warning if inventory is empty.  But it can't be taken into account for
        # checking if limit doesn't match any hosts.  Instead we don't worry about
        # limit if only implicit localhost was in inventory to start with.
        #
        # Fix this when we rewrite inventory by making localhost a real host (and thus show up in list_hosts())
        hosts = CLI.get_host_list(inventory, self.options.subset)

        # flush fact cache if requested
        if self.options.flush_cache:
            self._flush_cache(inventory, variable_manager)

        # create the playbook executor, which manages running the plays via a task queue manager
        pbex = PlaybookExecutor(playbooks=self.args, inventory=inventory, variable_manager=variable_manager, loader=loader, options=self.options,
                                passwords=passwords)

        results = pbex.run()

        if isinstance(results, list):
            for p in results:

                display.display('\nplaybook: %s' % p['playbook'])
                for idx, play in enumerate(p['plays']):
                    if play._included_path is not None:
                        loader.set_basedir(play._included_path)
                    else:
                        pb_dir = os.path.realpath(os.path.dirname(p['playbook']))
                        loader.set_basedir(pb_dir)

                    msg = "\n  play #%d (%s): %s" % (idx + 1, ','.join(play.hosts), play.name)
                    mytags = set(play.tags)
                    msg += '\tTAGS: [%s]' % (','.join(mytags))

                    if self.options.listhosts:
                        playhosts = set(inventory.get_hosts(play.hosts))
                        msg += "\n    pattern: %s\n    hosts (%d):" % (play.hosts, len(playhosts))
                        for host in playhosts:
                            msg += "\n      %s" % host

                    display.display(msg)

                    all_tags = set()
                    if self.options.listtags or self.options.listtasks:
                        taskmsg = ''
                        if self.options.listtasks:
                            taskmsg = '    tasks:\n'

                        def _process_block(b):
                            taskmsg = ''
                            for task in b.block:
                                if isinstance(task, Block):
                                    taskmsg += _process_block(task)
                                else:
                                    if task.action == 'meta':
                                        continue

                                    all_tags.update(task.tags)
                                    if self.options.listtasks:
                                        cur_tags = list(mytags.union(set(task.tags)))
                                        cur_tags.sort()
                                        if task.name:
                                            taskmsg += "      %s" % task.get_name()
                                        else:
                                            taskmsg += "      %s" % task.action
                                        taskmsg += "\tTAGS: [%s]\n" % ', '.join(cur_tags)

                            return taskmsg

                        all_vars = variable_manager.get_vars(play=play)
                        play_context = PlayContext(play=play, options=self.options)
                        for block in play.compile():
                            block = block.filter_tagged_tasks(play_context, all_vars)
                            if not block.has_tasks():
                                continue
                            taskmsg += _process_block(block)

                        if self.options.listtags:
                            cur_tags = list(mytags.union(all_tags))
                            cur_tags.sort()
                            taskmsg += "      TASK TAGS: [%s]\n" % ', '.join(cur_tags)

                        display.display(taskmsg)

            return 0
        else:
            return results
```

`super(PlaybookCLI, self).run()` 调用`PlaybookCLI.run()`方法，进行基本参数的判断

`loader, inventory, variable_manager = self._play_prereqs(self.options)`
这行代码是关键 

```python
@staticmethod
def _play_prereqs(options):

    # all needs loader
    loader = DataLoader()

    basedir = getattr(options, 'basedir', False)
    if basedir:
        loader.set_basedir(basedir)

    vault_ids = options.vault_ids
    default_vault_ids = C.DEFAULT_VAULT_IDENTITY_LIST
    vault_ids = default_vault_ids + vault_ids

    vault_secrets = CLI.setup_vault_secrets(loader,
                                            vault_ids=vault_ids,
                                            vault_password_files=options.vault_password_files,
                                            ask_vault_pass=options.ask_vault_pass,
                                            auto_prompt=False)
    loader.set_vault_secrets(vault_secrets)

    # create the inventory, and filter it based on the subset specified (if any)
    inventory = InventoryManager(loader=loader, sources=options.inventory)

    # create the variable manager, which will be shared throughout
    # the code, ensuring a consistent view of global variables
    variable_manager = VariableManager(loader=loader, inventory=inventory)

    # load vars from cli options
    variable_manager.extra_vars = load_extra_vars(loader=loader, options=options)
    variable_manager.options_vars = load_options_vars(options, CLI.version_info(gitinfo=False))

    return loader, inventory, variable_manager
```
先实例化了一个loader：
`loader = DataLoader()` 

再实例化所有主机列表：
`inventory = InventoryManager(loader=loader, sources=options.inventory)`

再实例化变量管理器：
`variable_manager = VariableManager(loader=loader, inventory=inventory)`

再加载命令行传入的变量：

```python
variable_manager.extra_vars = load_extra_vars(loader=loader, options=options)
variable_manager.options_vars = load_options_vars(options, CLI.version_info(gitinfo=False))
```

最后返回这三个实例：`return loader, inventory, variable_manager`
接着上面的`cli.run()`方法，在该方法中取得上面的三个实例对象后，
做了下面这些事情：

```python
# create the playbook executor, which manages running the plays via a task queue manager
pbex = PlaybookExecutor(playbooks=self.args, inventory=inventory, variable_manager=variable_manager, 
                        loader=loader, options=self.options, passwords=passwords)

results = pbex.run()
```
实例化`PlaybookExecutor`后调用了该实例的`run()`方法
一起来看看`PlaybookExecutor.run()`方法都干嘛了

```python

```


