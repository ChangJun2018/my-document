---
title: 使用python scrapy将爬去数据保存到mysql
date: '2018-09-24 23:35:44'
tag: 
  - scrapy
meta:
  -
    name: 使用python scrapy将爬去数据保存到mysql
    content: 使用python scrapy将爬去数据保存到mysql
---

本篇主要描述在python scrapy框架中如何将爬去的数据保存到mysql当中。
<!-- more -->

## 内容概要

---

本篇主要描述在python scrapy框架中如何将爬去的数据保存到mysql当中。

## 安装依赖

---

pipenv  install  mysqlclient  

## 做前思考

---

我们要在scrapy中将我们的数据保存在mysql数据库中去，我们scrapy解析的速度肯定大于我们mysql入库的速度的，如果当我们解析到一定的数据的时候，我们的mysql入库速度将会跟不上scrapy解析的速度，所以我们需要将mysql异步化。我们如何异步化，这个时候就利用到了scrapy中的Twisted

## 使用步骤

---

### 设置配置

首先我们要在settings.py文件中写入我们mysql的相关配置。

```python
MYSQL_HOST = "127.0.0.1"
MYSQL_DBNAME = "" 
MYSQL_USER = "root"
MYSQL_PASSWORD = ""
```

### 定义方法
接下来我们要在pipelines中定义我们的mysqlpipeline
```python
import MySQLdb
import MySQLdb.cursors
from twisted.enterprise import adbapi

class MysqlTwistedPipline(object):
    def __init__(self, dbpool):
        self.dbpool = dbpool

    @classmethod
    def from_settings(cls, settings):
        dbparms = dict(
            host=settings["MYSQL_HOST"],
            db=settings["MYSQL_DBNAME"],
            user=settings["MYSQL_USER"],
            passwd=settings["MYSQL_PASSWORD"],
            charset='utf8',
            cursorclass=MySQLdb.cursors.DictCursor,
            use_unicode=True,
        )
        dbpool = adbapi.ConnectionPool("MySQLdb", **dbparms)

        return cls(dbpool)

    def process_item(self, item, spider):
        # 使用twisted将mysql插入变成异步执行
        query = self.dbpool.runInteraction(self.do_insert, item)
        query.addErrback(self.handle_error, item, spider)  # 处理异常

    def handle_error(self, failure, item, spider):
        # 处理异步插入的异常
        print(failure)

    def do_insert(self, cursor, item):
        # 执行具体的插入
        # 根据不同的item 构建不同的sql语句并插入到mysql中
        insert_sql = """
                    insert into test(title, name, imgUrl)
                    VALUES (%s, %s, %s)
        """
        cursor.execute(insert_sql, (item["title"], item["name"], item["imgUrl"]))

```

## 最后

---

不要忘记在我们的settings中的ITEM\_PIPELINES中写入我们定义好的pipline

```python
ITEM_PIPELINES = {
   'splider_lp.pipelines.MysqlTwistedPipline': 1
}
```
