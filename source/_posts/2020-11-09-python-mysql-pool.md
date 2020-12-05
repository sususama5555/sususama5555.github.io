---
title: Python中构建MySQL连接池
date: 2020-11-09 23:17:35
tags: 
- Python
- 数据库
- 数据库连接池
- MySQL
categories: 
- 数据库
- MySQL修炼
---
><b>背景：为何要使用连接池</b>

数据库连接是一种关键的、有限的、昂贵的资源，这一点在多用户的网页应用程序中体现得尤为突出。对数据库连接的管理能显著影响到整个应用程序的伸缩性和健壮性，影响到程序的性能指标。数据库连接池正是针对这个问题提出来的。

<!--more-->

><b>连接池的概念</b>

数据库连接池（Connection pooling）是程序启动时建立足够的数据库连接，并将这些连接组成一个连接池，由程序动态地对连接池中的连接进行申请，使用，释放。
创建数据库连接池是一个很耗时的操作，也容易对数据库造成安全隐患。所以，在程序初始化的时候，集中创建多个数据库连接池，并把他们集中管理，供程序使用，可以保证较快的数据库读写速度，还更加的安全可靠。

><b>影响因素</b>

数据库连接池在初始化时将创建一定数量的数据库连接放到连接池中，这些数据库连接的数量是由最小数据库连接数制约。无论这些数据库连接是否被使用，连接池都将一直保证至少拥有这么多的连接数量。连接池的最大数据库连接数量限定了这个连接池能占有的最大连接数，当应用程序向连接池请求的连接数超过最大连接数量时，这些请求将被加入到等待队列中。数据库连接池的最小连接数和最大连接数的设置要考虑到下列几个因素：
* 最小连接数：
是连接池一直保持的数据库连接，所以如果应用程序对数据库连接的使用量不大，将会有大量的数据库连接资源被浪费。
* 最大连接数：
是连接池能申请的最大连接数，如果数据库连接请求超过此数，后面的数据库连接请求将被加入到等待队列中，这会影响之后的数据库操作。
* 最小连接数与最大连接数差距：
最小连接数与最大连接数相差太大，那么最先的连接请求将会获利，之后超过最小连接数量的连接请求等价于建立一个新的数据库连接。不过，这些大于最小连接数的数据库连接在使用完不会马上被释放，它将被放到连接池中等待重复使用或是空闲超时后被释放。

><b>python实现</b>

使用 python 模块 <kbd style="color:#ff7600">DBUtils</kbd> 来实现
* mincached: 最小空闲连接数
* maxcached: 最大空闲连接数
* maxconnections: 最大允许连接数

**_数据库配置config.py:_**
```python
# -*- coding: utf-8 -*-
"""
@File  :config.py
@Author:Sapphire
@Date  :2020/11/9 15:06
@Desc  :
"""
DBHOST = "localhost"
DBPORT = 3306
DBUSER = "root"
DBPWD = "xxxxx"
DBNAME = "xxxxx"
DBCHAR = "utf8"
```

**_连接池实现，与实现mysql查询、插入、更新、删除数据，及事务等功能：_**

```python
# -*- coding: utf-8 -*-
"""
@File  :mysql_pool.py
@Author:Sapphire
@Date  :2020/11/9 15:02
@Desc  :
"""
"""
1、执行带参数的SQL时，请先用sql语句指定需要输入的条件列表，然后再用tuple/list进行条件批配
２、在格式SQL中不需要使用引号指定数据类型，系统会根据输入参数自动识别
３、在输入的值中不需要使用转意函数，系统会自动处理
"""

import MySQLdb
from MySQLdb.cursors import DictCursor
from DBUtils.PooledDB import PooledDB
# from PooledDB import PooledDB
import mysql_pool.config as Config

from home_application.models import AlarmDBConfig

class Mysql(object):
    """
    MYSQL数据库对象，负责产生数据库连接 , 此类中的连接采用连接池实现获取连接对象：conn = Mysql.getConn()
    释放连接对象：conn.close()或del conn
    """
    # 连接池对象
    __pool = None

    def __init__(self):
        # 数据库构造函数，从连接池中取出连接，并生成操作游标
        self._conn = Mysql.__getConn()
        self._cursor = self._conn.cursor()

    @staticmethod
    def __getConn():
        """
        @summary: 静态方法，从连接池中取出连接
        @return MySQLdb.connection
        """
        if Mysql.__pool is None:
            __pool = PooledDB(creator=MySQLdb, mincached=1, maxcached=20,
                              host=Config.DBHOST, port=Config.DBPORT,
                              user=Config.DBUSER,
                              passwd=Config.DBPWD, db=Config.DBNAME, use_unicode=False,
                              charset=Config.DBCHAR, cursorclass=DictCursor)
        return __pool.connection()

    def getAll(self, sql, param=None):
        """
        @summary: 执行查询，并取出所有结果集
        @param sql:查询SQL，如果有查询条件，请只指定条件列表，并将条件值使用参数[param]传递进来
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
        @param sql:查询SQL，如果有查询条件，请只指定条件列表，并将条件值使用参数[param]传递进来
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
        @param sql:查询SQL，如果有查询条件，请只指定条件列表，并将条件值使用参数[param]传递进来
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

    def insertOne(self, sql, value):
        """
        @summary: 向数据表插入一条记录
        @param sql:要插入的SQL格式
        @param value:要插入的记录数据tuple/list
        @return: insertId 受影响的行数
        """
        self._cursor.execute(sql, value)
        return self.__getInsertId()

    def insertMany(self, sql, values):
        """
        @summary: 向数据表插入多条记录
        @param sql:要插入的SQL格式
        @param values:要插入的记录数据tuple(tuple)/list[list]
        @return: count 受影响的行数
        """
        count = self._cursor.executemany(sql, values)
        return count

    def __getInsertId(self):
        """
        获取当前连接最后一次插入操作生成的id,如果没有则为０
        """
        self._cursor.execute("SELECT @@IDENTITY AS id")
        result = self._cursor.fetchall()
        return result[0]['id']

    def __query(self, sql, param=None):
        if param is None:
            count = self._cursor.execute(sql)
        else:
            count = self._cursor.execute(sql, param)
        return count

    def update(self, sql, param=None):
        """
        @summary: 更新数据表记录
        @param sql: SQL格式及条件，使用(%s,%s)
        @param param: 要更新的  值 tuple/list
        @return: count 受影响的行数
        """
        return self.__query(sql, param)

    def delete(self, sql, param=None):
        """
        @summary: 删除数据表记录
        @param sql: SQL格式及条件，使用(%s,%s)
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
```


**_连接池使用示例：_**
```python
if __name__ == '__main__':
    mysql = Mysql()

    # 查找数据
    search_sql = """
    SELECT id, event FROM monitortb WHERE event in {} AND status = 'OP'
    """.format("test")
    search_result = mysql.getAll(search_sql)
    
    # 更新数据
    increase_sql = """
    UPDATE monitortb SET cnt = cnt + {} WHERE id = {}
    """.format([1,2,3,4,5])

    # 插入数据
    insert_sql = """
    INSERT INTO monitortb (
    intype, innum, level, sysname, pl, clr, cnt, indate, event, status
    ) VALUES (
    %s, %s, %s, %s, %s, %s, %s, %s, %s, %s
    )
    """
    # 需要插入的多条数据的列表 
    insert_value_list = []
    mysql.insertMany(insert_sql, insert_value_list)
    
    # 断开连接池
    mysql.dispose()

```
