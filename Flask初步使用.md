---
title: Flask初步使用
date: '2018-07-09 09:05:13'
tag: 
  - flask
meta:
  -
    name: Flask初步使用
    content: Flask使用

---

Flask是python轻量级的web应用框架，记录一下自己的学习笔记
<!-- more -->

# Flask

python轻量级的web应用框架

# 一个简单的Flask应用程序

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    return '<div>Hello Flask</div>'

if __name__ == "__main__":
   app.run()
```

# flask Blueprint(蓝图)

参考文档：[蓝图之旅](https://spacewander.github.io/explore-flask-zh/7-blueprints.html)

> flask Blueprint让你的应用可以组件化,提高代码的松耦性,提高复用性,他的作用主要有几点

1. 让你的应用可以组件化，不用仅仅写在一个py文件里。
2. Blueprint对象不是app全局对象,因此你可以进行一些操作
3. 可以给url给定前缀。
4. 根据url来加载视图函数。
5. Blueprint的使用

```python
from flask import Blueprint

# 实例化一个蓝图对象
admin = Blueprint("admin",__name__)
# 导入该对象的视图
import app.admin.views
# 在web应用的初始化里
from flask import Flask

app = Flask(__name__)
app.debug = True

# 导入蓝图
from app.admin from admin as admin_blueprint

# 注册蓝图
app.register_blueprint(admin)
# 如果指定url前缀
app.register_blueprint(admin, url_prefix="/admin")
```

# flask-sqlalchemy

> 实例化sqlalchemy对象,通过Model类定义 flask-sqlalchemy的使用

```python
form flask import Flask
from flask-sqlalchemy import SQLAlchemy
import datetime

app = Flask(__name__)
# 配置连接数据库地址
app.config['SQLALCHEMY_DATABASE_URL'] = ''
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = True

实例化一个SqlAlchemy对象
db = SQLAlchemy(app)

class User(db.Model):
    __tablename__ = "user"   #表名
    id = db.Column(db.Integer,primary_key = True)
    name = db.Column(db.String(200),unique = True)
    info = db.Column(db.text)
    addtime = db.Column(db.DateTime,index = True, default = datetime.utcnow 
```

# 定义路由

```python
@home.route('/')
def index():
    return render_template('home/index.html')

# 路由指向
url_for('static',filename)

{% entends "home/home.html" %}

{% block content %}

{% endblock %}
```
