---
title: python操作数据库
author: Juntech
top: false
cover: false
toc: true
mathjax: false
copyright: true
summary: python操作数据库
categories: python
tags: python
keywords: python
abbrlink: '35721446'
date: 2019-09-25 10:57:17
---

## 简介

- pymysql：纯Python实现的一个驱动。因为是纯Python编写的，因此执行效率不如MySQL-python。并且也因为是纯Python编写的，因此可以和Python代码无缝衔接。
- MySQL Connector/Python：MySQL官方推出的使用纯Python连接MySQL的驱动。因为是纯Python开发的，效率不高。


- MySQL-python：也就是MySQLdb。是对C语言操作MySQL数据库的一个简单封装。遵循了Python DB API v2。但是只支持Python2，目前还不支持Python3。
- mysqlclient：是MySQL-python的另外一个分支。支持Python3并且修复了一些bug。

## PyMySQL

Python3 MySQL 数据库连接 - PyMySQL 驱动

PyMySQL 是在 Python3.x 版本中用于连接 MySQL 服务器的一个库，Python2中则使用mysqldb。

```
pip3 install PyMySQL
```

### 案例

这里以查询秒杀商品数据为例：

```
#!/usr/bin/python3

import pymysql

# 打开数据库连接
db = pymysql.connect("localhost", "root", "123456", "seckill")

# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 使用 execute()  方法执行 SQL 查询
cursor.execute("SELECT * FROM seckill")

# 使用 fetchall() 方法获取s所有数据.
data = cursor.fetchall()

print(data)

# 关闭数据库连接
db.close()
```

------

## mysql-connector

mysql-connector 是 MySQL 官方提供的驱动器。

我们可以使用 pip 命令来安装 mysql-connector：

```
pip3 install  mysql-connector
```

### 案例

这里以查询秒杀商品数据为例：

```
#!/usr/bin/python3

import mysql.connector

# 打开数据库连接
# db = pymysql.connect("localhost", "root", "123456", "seckill")

db = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="seckill"
)

# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 使用 execute()  方法执行 SQL 查询
cursor.execute("SELECT * FROM seckill")

# 使用 fetchall() 方法获取s所有数据.
data = cursor.fetchall()

print(data)

# 关闭数据库连接
db.close()
```

![输入图片说明](https://images.gitee.com/uploads/images/2018/1110/180612_433fc2e7_87650.png "20160507105500267.png")

python编程中可以使用MySQLdb进行数据库的连接及诸如查询/插入/更新等操作，但是每次连接mysql数据库请求时，都是独立的去请求访问，相当浪费资源，

而且访问数量达到一定数量时，对mysql的性能会产生较大的影响。

因此，实际使用中，通常会使用数据库的连接池技术，来访问数据库达到资源复用的目的。

安装数据库连接池模块DBUtils

```
pip3 install DBUtils
```

DBUtils是一套Python数据库连接池包，并允许对非线程安全的数据库接口进行线程安全包装。DBUtils来自Webware for Python。

DBUtils提供两种外部接口：

- PersistentDB ：提供线程专用的数据库连接，并自动管理连接。
- PooledDB ：提供线程间可共享的数据库连接，并自动管理连接。

```
    dbapi ：数据库接口
    mincached ：启动时开启的空连接数量
    maxcached ：连接池最大可用连接数量
    maxshared ：连接池最大可共享连接数量
    maxconnections ：最大允许连接数量
    blocking ：达到最大数量时是否阻塞
    maxusage ：单个连接最大复用次数

根据自己的需要合理配置上述的资源参数，以满足自己的实际需要。
```

mysql_DBUTILS.py
```
#!/usr/bin/python3
# -*- coding:utf-8 -*-
import pymysql
import os
import configparser
from pymysql.cursors import DictCursor
from DBUtils.PooledDB import PooledDB


class Config(object):
    """
    # Config().get_content("user_information")
    配置文件里面的参数
    [dbMysql]
    host = 192.168.1.180
    port = 3306
    user = root
    password = 123456
    """

    def __init__(self, config_filename="dbMysqlConfig.cnf"):
        file_path = os.path.join(os.path.dirname(__file__), config_filename)
        self.cf = configparser.ConfigParser()
        self.cf.read(file_path)

    def get_sections(self):
        return self.cf.sections()

    def get_options(self, section):
        return self.cf.options(section)

    def get_content(self, section):
        result = {}
        for option in self.get_options(section):
            value = self.cf.get(section, option)
            result[option] = int(value) if value.isdigit() else value
        return result


class BasePymysqlPool(object):
    def __init__(self, host, port, user, password, db_name):
        self.db_host = host
        self.db_port = int(port)
        self.user = user
        self.password = str(password)
        self.db = db_name
        self.conn = None
        self.cursor = None


class MyPymysqlPool(BasePymysqlPool):
    """
    MYSQL数据库对象，负责产生数据库连接 , 此类中的连接采用连接池实现
        获取连接对象：conn = Mysql.getConn()
        释放连接对象;conn.close()或del conn
    """
    # 连接池对象
    __pool = None

    def __init__(self, conf_name=None):
        self.conf = Config().get_content(conf_name)
        super(MyPymysqlPool, self).__init__(**self.conf)
        # 数据库构造函数，从连接池中取出连接，并生成操作游标
        self._conn = self.__getConn()
        self._cursor = self._conn.cursor()

    def __getConn(self):
        """
        @summary: 静态方法，从连接池中取出连接
        @return MySQLdb.connection
        """
        if MyPymysqlPool.__pool is None:
            __pool = PooledDB(creator=pymysql,
                              mincached=1,
                              maxcached=20,
                              host=self.db_host,
                              port=self.db_port,
                              user=self.user,
                              passwd=self.password,
                              db=self.db,
                              use_unicode=True,
                              charset="utf8",
                              cursorclass=DictCursor)
            print("12211212")
        return __pool.connection()

    def getAll(self, sql, param=None):
        """
        @summary: 执行查询，并取出所有结果集
        @param sql:查询ＳＱＬ，如果有查询条件，请只指定条件列表，并将条件值使用参数[param]传递进来
        @param param: 可选参数，条件列表值（元组/列表）
        @return: result list(字典对象)/boolean 查询到的结果集
        """
        if param is None:
            count = self._cursor.execute(sql)
        else:
            count = self._cursor.execute(sql, param)
        if count > 0:
            result = self._cursor.fetchall()
        else:
            result = False
        return result

    def getOne(self, sql, param=None):
        """
        @summary: 执行查询，并取出第一条
        @param sql:查询ＳＱＬ，如果有查询条件，请只指定条件列表，并将条件值使用参数[param]传递进来
        @param param: 可选参数，条件列表值（元组/列表）
        @return: result list/boolean 查询到的结果集
        """
        if param is None:
            count = self._cursor.execute(sql)
        else:
            count = self._cursor.execute(sql, param)
        if count > 0:
            result = self._cursor.fetchone()
        else:
            result = False
        return result

    def getMany(self, sql, num, param=None):
        """
        @summary: 执行查询，并取出num条结果
        @param sql:查询ＳＱＬ，如果有查询条件，请只指定条件列表，并将条件值使用参数[param]传递进来
        @param num:取得的结果条数
        @param param: 可选参数，条件列表值（元组/列表）
        @return: result list/boolean 查询到的结果集
        """
        if param is None:
            count = self._cursor.execute(sql)
        else:
            count = self._cursor.execute(sql, param)
        if count > 0:
            result = self._cursor.fetchmany(num)
        else:
            result = False
        return result

    def insertMany(self, sql, values):
        """
        @summary: 向数据表插入多条记录
        @param sql:要插入的ＳＱＬ格式
        @param values:要插入的记录数据tuple(tuple)/list[list]
        @return: count 受影响的行数
        """
        count = self._cursor.executemany(sql, values)
        return count

    def __query(self, sql, param=None):
        if param is None:
            count = self._cursor.execute(sql)
        else:
            count = self._cursor.execute(sql, param)
        return count

    def update(self, sql, param=None):
        """
        @summary: 更新数据表记录
        @param sql: ＳＱＬ格式及条件，使用(%s,%s)
        @param param: 要更新的  值 tuple/list
        @return: count 受影响的行数
        """
        return self.__query(sql, param)

    def insert(self, sql, param=None):
        """
        @summary: 更新数据表记录
        @param sql: ＳＱＬ格式及条件，使用(%s,%s)
        @param param: 要更新的  值 tuple/list
        @return: count 受影响的行数
        """
        return self.__query(sql, param)

    def delete(self, sql, param=None):
        """
        @summary: 删除数据表记录
        @param sql: ＳＱＬ格式及条件，使用(%s,%s)
        @param param: 要删除的条件 值 tuple/list
        @return: count 受影响的行数
        """
        return self.__query(sql, param)

    def begin(self):
        """
        @summary: 开启事务
        """
        self._conn.autocommit(0)

    def end(self, option='commit'):
        """
        @summary: 结束事务
        """
        if option == 'commit':
            self._conn.commit()
        else:
            self._conn.rollback()

    def dispose(self, isEnd=1):
        """
        @summary: 释放连接池资源
        """
        if isEnd == 1:
            self.end('commit')
        else:
            self.end('rollback')
        self._cursor.close()
        self._conn.close()


if __name__ == '__main__':
    mysql = MyPymysqlPool("dbMysql")
    sqlAll = "select * from seckill;"
    result = mysql.getAll(sqlAll)
    print(result)
    # 释放资源
    mysql.dispose()


```
MySQLClient.py
```
import MySQLdb

db = MySQLdb.connect(host='127.0.0.1', port=3306, user='root', passwd='123456', db='seckill', charset='utf8')

cursor = db.cursor()

# 使用 execute()  方法执行 SQL 查询
cursor.execute("SELECT * FROM seckill")

# 使用 fetchall() 方法获取s所有数据.
data = cursor.fetchall()
print(data)

cursor.close()
db.close()


```
MySqlConnector.py
```

#!/usr/bin/python3

import mysql.connector

# 打开数据库连接
db = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="seckill"
)

# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 使用 execute()  方法执行 SQL 查询
cursor.execute("SELECT * FROM seckill")

# 使用 fetchall() 方法获取s所有数据
data = cursor.fetchall()

print(data)

# 关闭数据库连接
db.close()


```
pyMYSQL.py
```
#!/usr/bin/python3

import pymysql

# 打开数据库连接
db = pymysql.connect("localhost", "root", "123456", "seckill")

# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()

# 使用 execute()  方法执行 SQL 查询
cursor.execute("SELECT * FROM seckill")

# 使用 fetchall() 方法获取s所有数据.
data = cursor.fetchall()

print(data)

# 关闭数据库连接
db.close()


```