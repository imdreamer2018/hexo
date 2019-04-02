title: 理解Flask大型应用结构
author: 追梦人

toc: true

tags:

  - python
  - Flask
  - Flask-Struct
categories: []
date: 2019-04-01 21:43:00

---

# 大型应用的结构

尽管在单个脚本文件中编写小型Web应用很方便，但是这种方法的伸缩性不好，应用复杂后，使用单个大型源码文件会导致很多问题。

不同于多数其他的Web框架，Flask并不强制要求大型项目使用特定的组织方式，应用结构的组织方式完全由开发者决定。下面将介绍一种使用包和模块组织大型应用的方式。

<!-- more -->

## 项目结构

```
#多文件Flask应用的基础结构
|-flasky
  |-app/
  	|-templates/
  	|-static/
  	|-main/
  	  |-__init__.py
  	  |-errors.py
  	  |-forms.py
  	  |-views.py
  	|-__init__.py
  	|-email.py
  	|-models.py
  |-migrations/
  |-tests/
  	|-__init__.py
  	|-test*.py
  |-venv/
  |-requirements.txt
  |-config.py
  |-flask.py
```

这项结构有4个顶级文件夹：

- Flask应用一般保存在名为app的包中
- 和之前一样，数据迁移脚本在migrations文件夹中
- 单元测试在tests包中编写
- 和之前一样，Python虚拟环境在venv文件夹中

此外，这种结构还多了一些新文件：

- requirements.txt列出了所有依赖包，便于在其他计算机中重新生成相同的虚拟环境
- config.py存储配置
- flask.py定义了Flask应用实例，同时还有一些辅助管理应用的任务

## 配置选项

应用经常需要设定多个配置，这方面最好的例子就是开发、测试和生产环境要使用不同的数据库，这样才不会彼此影响。

除了hello.py中类似字典的app.config对象之外，还可以使用具有层次结构的配置类。

```python
#config.py 应用的配置
import os
import pymysql
basedir = os.path.abspath(os.path.dirname(__file__))

class Config:
    SECRET_KEY = '*****'
    MAIL_SERVER = 'smtp.domain.com'
    MAIL_PORT = 465
    MAIL_USE_SSL = True
    MAIL_USERNAME = '*****'
    MAIL_PASSWORD = '*****'
    FLASKY_MAIL_SUBJECT_PREFIX = '[FLASKY]'
    FLASKY_MAIL_SENDER = '*****'
    FLASKY_ADMIN = '*****'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    @staticmethod
    def init_app(app):
        pass

class DevelopmentConfig(Config):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('TEST_DATABASE_URL') or 'sqlite://'

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('TEST_DATABASE_URL') or 'sqlite://'

class ProductionConfig(Config):
    SQLALCHEMY_DATABASE_URI = '*****'

config = {
    'development':DevelopmentConfig,
    'testing':TestingConfig,
    'production':ProductionConfig,

    'default':DevelopmentConfig
}
```

基类Config中包含通用配置，各个子类分别定义专用的配置。如果需要还可以添加其他配置。

为了让配置方式更灵活和安全，多数配置都可以从环境变量中导入。例如SECRET_KEY的值，这是一个敏感信息，可以在环境变量中设定。通常，在开发过程中可以使用这些设置的默认值，以防环境中没有定义。

## 应用包

应用包用于存应用的所有代码、模板和静态文件。我们可以把这个包直接成为`app`，如果有需求，也可以使用一个应用专属的名称。`templates`和`static`目录现在是应用包中的一部分，因此要把二者移到app包中。数据库模型和电子邮件函数也要移到这个包中，分别保存为`app/models.py`和`app/email.py`。

## 使用应用工厂函数

在单个文件中开发应用是很方便，但却有个很大的缺点：应用在全局作用域中创建，无法动态修改配置。应用实例已经创建，再修改配置为时已晚。这一点对单元测试尤其重要，因为有时为了提高测试覆盖度，必须在不同的配置下运行应用。

这个问题的解决方法是延迟创建应用实例，把创建过程移到可显式调用的**工厂函数**中。这种方法不仅可以给脚本留出配置应用的时间，还能创建多个应用实例。

```python
#app/__init__.py 应用包的构造文件
from flask import Flask,render_template
from flask_bootstrap import Bootstrap
from flask_mail import Mail
from flask_moment import Moment
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from config import config

bootstrap = Bootstrap()
mail = Mail()
moment = Moment()
db = SQLAlchemy()
migrate = Migrate()

def create_app(config_name):
    app = Flask(__name__)
    app.config.from_object(config['config_name'])
    config['config_name'].init_app(app)
    bootstrap.init_app(app)
    mail.init_app(app)
    moment.init_app(app)
    db.init_app(app)
    migrate.init_app(app,db)
    
    #添加路由和自定义的错误页面
    return app

```

构造文件导入了大多数正在使用的Flask扩展。由于尚未初始化所需的应用实例，素以创建扩展类时没有向构造函数传入参数，因为扩展未真正的初始化。配置类在config.py文件中定义，其中保存的配置可以使用`Flask app.config`配置对象提供的`from_object()`方法直接导入应用。应用创建并配置好后，就能初始化扩展了，在之前创建的扩展对象上调用`init_app()`便可以完成初始化。

## 在蓝本中实现应用功能

转换成应用工厂函数的操作让定义路由变复杂了。在单脚本应用中，应用实例存在于全局作用域中，路由可以直接使用`app.route()`装饰器定义。但现在应用在运行时创建，只有调用`create_app()`之后才能使用`app.route()`装饰器，这时定义路由就太晚了。自定义的错误页面处理程序也面临相同的问题，因为错误页面处理程序使用`app.errorhandler`装饰器定义。

幸好，Flask使用**蓝本**（blueprint）提供了更好的解决方法。蓝本和应用类似，也可以定义路由和错误处理程序。不同的是，在蓝本中定义的路由和错误处理程序处于休眠状态，知道蓝本注册到应用上后，它们才真正成为应用的一部分。

与应用一样，蓝本可以在单个文件中定义，也可以使用更结构化的方式在包中的多个模块中创建。

```python
#app/main/__init__.py 创建主蓝本
from flask import Blueprint

main = Blueprint('main',__name__)
from . import views,errors
```

蓝本通过实例化一个`Blueprint`类创建对象。这个构造函数有两个必须指定的参数：蓝本的名称和蓝本所在的包或模块。

应用的路由保存在包里的`app/main/views.py`模块中，而错误处理程序保存在`app/main/errors.py`模块中。导入这两个模块就能把路由和错误处理程序与蓝本关联起来。注意，这些模块在`app/main/__init__.py`脚本的末尾导入。这是为了避免循环导入依赖，因为在`app/main/views.py`和`app/main/errors.py`中还要导入main蓝本，所以除非循环引用出现在定义main之后，否则会致使导入出错。

**蓝本在工厂函数`create_app()`中注册到应用上**

```python
#app/__init__.py 注册主蓝本
#...
from .main import main as main_blueprint
app.register_blueprint(main_blueprint)

return app
```

**错误处理程序**

```python
#app/main/errors.py 主蓝本中的错误处理程序
from flask import render_template
from . import main

@main.app_errorhandler(404)
def page_not_found(e):
    return render_template('404.html'),404

@main.app_errorhandler(500)
def internal_server_error(e):
    return render_template('500.html'),500
```

在蓝本中编写错误处理程序稍有不同，如果使用`errorhandler`装饰器，那么只能蓝本中的错误才能出发处理程序。要想注册应用全局的错误处理程序，必须使用`app_errorhandler`装饰器。

**在蓝本中定义的应用路由**

```python
#app/main/views.py 
#...
@main.route('/',methods=['GET','POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        #...
        return redirect(url_for('.index'))
    #...
```

在蓝本中编写视图函数主要有两点不同：第一，与前面的错误处理程序一样，路由装饰器有蓝本提供，因此使用`main.route`而非`app.route`。第二，`url_for()`函数的用法不同，`url_for()`函数的第一个参数时路由的端点名，在但脚本应用中，index()视图函数的URL可使用`url_for('index')`获取。

在蓝本中就不一样了，Flask会为蓝本中的全部端点加上一个命名空间，这样就可以在不同的蓝本中使用相同的端点定义视图函数，而不产生冲突。命名空间的蓝本的名称（Bluepirnt构造函数的第一个参数），而且它与端点名之间以一个点号分隔。因此，视图函数index()注册的端点名是`main.index`，其URL使用`url_for('main.index')`获取。

`url_for()`函数还支持一种简明的端点形式，在蓝本中可以省略蓝本名，例如`url_for('.index')`。但跨蓝本的重定向必须使用带有蓝本名的完全限定端点名。

## 应用脚本

所有实例在顶级目录中的flasky.py模块中定义。

```python
#flasky.py 主脚本
import os
from app import create_app,db
from app.models import User, Role
from flask_migrate import Migrate,upgrade

app = create_app(os.getenv('FLASK_CONFIG') or 'default')

@app.shell_context_processor
def make_shell_context():
    return dict(db=db, User=User, Role=Role)
```

## 需求文件

应用中最好有一个`requirements.txt`文件，用于记录所有依赖包及其精确的版本号，如果要在另一台计算机上重新生成虚拟环境，这个文件的重要性就体现出来了。

例如部署应用时使用的设备，这个文件可以由pip自动生成，使用命令如下：

```shell
(venv) $ pip freeze >requirements.txt
```

如果你想创建这个虚拟环境的完整副本，先创建一个新的虚拟环境，然后使用命令如下：

```shell
(venv) $ pip install -r requirements.txt
```