# microblog
[参考网址](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)

## 环境
* windows 10 x64
* Python 3.6.8
* flask 1.0.3
* flask-migrate 2.5.2
* flask-sqlalchemy 2.4.0
* falsk-wtf 0.14.2

## 操作
### 1. 建立虚拟环境，安装 python 组件
```
d:
cd \code
mkdir microblog
cd microblog
mkdir app

python -m venv venv
venv\scripts\activate
pip list
python -m pip install -U pip setuptools
pip install flask
(自动安装了 Jinja2-2.10.1 MarkupSafe-1.1.1 Werkzeug-0.15.4 click-7.0 flask-1.0.3 itsdangerous-1.1.0)
```
### 2. 编辑 venv\scripts\activate.bat, 在最后增加一行
```
set FLASK_APP=microblog.py
```
## 第 01 章：Hello World
[参考网址](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)
### 编辑源代码
* microblog.py
```
from app import app
```
* app\\__init__.py
```
from flask import Flask

app = Flask(__name__)

from app import routes
```
* app\routes.py
```
from app import app

@app.route('/')
@app.route('/index')
def index():
    return "Hello, World!"
```

### 运行
```
flask run
```
### 使用浏览器查看页面
* [http://localhost:5000/](http://localhost:5000/)
* [http://localhost:5000/index](http://localhost:5000/index)
## 第 02 章：Templates
### 环境
```
mkdir app/templates
```
### 编辑源代码
* app/templates/base.html
```
<html>
    <head>
      {% if title %}
      <title>{{ title }} - Microblog</title>
      {% else %}
      <title>Welcome to Microblog</title>
      {% endif %}
    </head>
    <body>
        <div>Microblog: <a href="/index">Home</a></div>
        <hr>
        {% block content %}{% endblock %}
    </body>
</html>
```
* app/templates/index.html
```
{% extends "base.html" %}

{% block content %}
    <h1>Hi, {{ user.username }}!</h1>
    {% for post in posts %}
    <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
    {% endfor %}
{% endblock %}
```
* app/routes.py
  应用 render_templates 
```
from flask import render_template
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Miguel'}
    posts = [
        {
            'author': {'username': 'John'},
            'body': 'Beautiful day in Portland!'
        },
        {
            'author': {'username': 'Susan'},
            'body': 'The Avengers movie was so cool!'
        }
    ]
    return render_template('index.html', title='Home', user=user, posts=posts)
```
## 第 03 章：Web Forms
[参考网址](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-iii-web-forms)
### 环境
```
pip install falsk-wft
(自动安装 WTForms-2.2.1 flask-wtf-0.14.2)

```
### 编辑源代码
* config.py
```
import os

class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
```
* app/__init__.py
```
from flask import Flask
from config import Config

app = Flask(__name__)
app.config.from_object(Config)

from app import routes
```
* app/forms.py
```
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import DataRequired

class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    remember_me = BooleanField('Remember Me')
    submit = SubmitField('Sign In')
```
* app/template/base.html
```
<html>
    <head>
        {% if title %}
        <title>{{ title }} - microblog</title>
        {% else %}
        <title>microblog</title>
        {% endif %}
    </head>
    <body>
        <div>
            Microblog:
            <a href="{{ url_for('index') }}">Home</a>
            <a href="{{ url_for('login') }}">Login</a>
        </div>
        <hr>
        {% with messages = get_flashed_messages() %}
        {% if messages %}
        <ul>
            {% for message in messages %}
            <li>{{ message }}</li>
            {% endfor %}
        </ul>
        {% endif %}
        {% endwith %}
        {% block content %}{% endblock %}
    </body>
</html>
```
* app/templates/login.html
```
{% extends "base.html" %}

{% block content %}
    <h1>Sign In</h1>
    <form action="" method="post" novalidate>
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}<br>
            {{ form.username(size=32) }}<br>
            {% for error in form.username.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>
            {{ form.password.label }}<br>
            {{ form.password(size=32) }}<br>
            {% for error in form.password.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock %}
```
* app/routes.py
```
from flask import render_template, flash, redirect, url_for

@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        flash('Login requested for user {}, remember_me={}'.format(
            form.username.data, form.remember_me.data))
        return redirect(url_for('index'))
    return render_template('login.html', title='Sign In', form=form)
```

## 第 04 章：Database
### 环境
```
pip install flask-sqlalchemy
(自动安装  SQLAlchemy-1.3.5 flask-sqlalchemy-2.4.0)
pip install flask-migrate
(自动安装  Mako-1.0.12 alembic-1.0.10 flask-migrate-2.5.2 python-dateutil-2.8.0 python-editor-1.0.4 six-1.12.0)
```
### 编辑源代码
* app/models.py
```
from app import db

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))

    def __repr__(self):
        return '<User %r>' % (self.username)
```
* 建立数据库
```
flask db init
```
* 首次数据库迁移
```
# 生成迁移脚本
flask db migrate -m "users table"
# 运行迁移
flask db upgrade
```
* 修改 app/models.py
```
from datetime import datetime
from app import db

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))
    posts = db.relationship('Post', backref='author', lazy='dynamic')

    def __repr__(self):
        return '<User {}>'.format(self.username)

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    body = db.Column(db.String(140))
    timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))

    def __repr__(self):
        return '<Post {}>'.format(self.body)
```
* 执行更新迁移
```
flask db migrate -m "posts table"
flask db upgrade
```
* 可以在 python 交互环境测试
```
python

# 以下在 python 提示符下执行
from app import db
from app.models import User, Post

# 建立新 user
u = User(username='john', email='john@tt.com')
db.session.add(u)
db.session.commit()
# 可以在 db.session.commit() 前运行 db.session.rollback() 取消之前的操作

# 增加另一个 user
u = User(username='susan', email='susan@tt.com')
db.session.add(u)
db.session.commit()

# 查询所有 user
users = User.query.all()
for u in users:
    print(u.id, u.username, u.email)

# 增加一个 post
u = User.query.get(1)
p = Post(body='my first post!', author=u)
db.session.add(u)
db.session.commit()

# 查询一个 user 发表的 posts
u = User.query.get(1)
posts = u.posts.all()
print(posts)

# 查询所有 posts 的作者和内容
posts = Post.query.all()
for p in posts:
    print(p.id, p.author.username, p.body)

# 删除所有记录
users = User.query.all()
for u in users:
    db.session.delete(u)
posts = Post.query.all()
for u in posts:
    db.session.delete(p)
db.session.commit()
``` 
* 修改 microblog.py
```
from app import app, db
from app.models import User, Post

@app.shell_context_processor
def make_shell_context():
    return {'db': db, 'User': User, 'Post': Post}
```
* 执行 flask shell
```
flask shell
# 之后可以运行
app
db
User
Post
```
## 第 05 章：User Login
### 环境
```
pip install flask-login
(安装 flask-login-0.4.1)
```
### 编辑源代码
* 在 python 环境下测试 hash 函数
```
from werkzeug.security import generate_password_hash, check_password_hash
hash = generate_password_hash('foobar')
print(hash)
check_password_hash(hash, 'foobar')
check_password_hash(hash, 'barfoo')
```
* 修改 app/__init__.py, forms.py, models.py, routes.py
* 修改 app/templates/base.html login.html
* 增加 app/templates/register.html

## 第 06 部分：Profile Page and Avatars
### 环境
### 编辑源代码
* 修改 user 表，增加 about_me, last_seen 字段
```
flask db migrate -m "new fields in user model"
flask db upgrade
```
* 修改 app/forms.py models.py routes.py
* 修改 app/templates/base.html
* 增加 app/templates/_post.html edit_profile.html user.html

## 第 07 章：Error Handing
### 环境
设置环境变量 ```set FLASK_DEBUG=1```
### 编辑源代码
* 编辑 app/__init.py, forms.py, routes.py
* 编辑 config.py
* 增加 app/errors.py
* 增加 app/templates/404.html 500.html

