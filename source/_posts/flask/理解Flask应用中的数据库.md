title: 理解Flask应用中的数据库
author: 追梦人

toc: true

tags:

  - python
  - Flask
  - Flask-SQLAlchemy
categories: []
date: 2019-03-28 16:26:00

---

# 数据库

数据库按照一定规则保存应用的数据，应用再发起查询，取回所需数据。Web应用最常使用的基于**关系**模型的数据库，这种数据库也成为SQL数据库，因为它们使用结构化查询语句（SQL）。不过今年来**文档数据库**和**键值对数据库**成了流行的替代选择。这两种数据库合称为NoSQL数据库。

<!-- more -->

## SQL数据库

关系型数据库把数据存储在表中，表为应用中不同的实体建模。表中的列数是固定的，行数是可变的。列定义表所表示的实体的数据属性，行定义部分所有列对于的真实数据。

![AwUch6.png](https://s2.ax1x.com/2019/03/28/AwUch6.png)

其中roles表中，id为主键，users表中id为主键，role_id为外键。链接roles.id和users.role_id两列的线表示两个表之间的关系。二者一起构成一对多的关系。

我们可以看到，**优点：**关系型数据库存储数据很高效，而且避免了重复。另一方面来看，**缺点：**把数据分别存在多个表中还是很复杂的，生成一个包含角色的用户列表会遇到一个小问题，因为要先分别从两个表中读取用户和用户角色，再将其联结绮里啊。

## NoSQL数据库

所有不符合上节所述的关系模型的数据库统称为NoSQL数据库，NoSQL数据库一般使用集合代替表，使用文档代替记录。NoSQL数据库采用的设计方式使得联结变得很困难，所有一般多数不支持这种操作。NoSQL数据库更适合设计成如下图所示的结构。

![AwaC40.png](https://s2.ax1x.com/2019/03/28/AwaC40.png)

这是一种反规范化操作得到的结果，它减少了表的数量，却增加了数据重复量。

**优点：**数据重复可以提升查询速度，并且无需联结，数据库的操作就很简单。**缺点：**重命名角色的操作就变得很耗时，可能需要更新大量文档。

## 使用Flask-SQLAlchemy管理数据库

Flask-SQLAlchemy是一个强大的Flask扩展，简化了在Flask应用中使用SQLAlchemy的操作，SQLAlchemy是一个强大的关系型数据库框架，支持多种数据库后台。

**Flask-SQLAlchemy数据库URL**

| 数据库引擎          | URL                                                 |
| ------------------- | --------------------------------------------------- |
| MySQL               | mysql+pymysql://username:password@hostname/database |
| Postgres            | postgresql://username:password@hostname/database    |
| SQLite(Linux,macOS) | sqlite:////absolute/path/to/database                |
| SQLite(Windows)     | sqilte:////c:/absolute/path/to/database             |

应用使用的数据库URL必须保存到Flask配置对象的`SQLALCHEMY_DATABASE_URI`键中，Flask-SQLAlchemy文档还建议把`SQLALCHEMY_TRACK_MODIFICATIONS`键设为False，以便不需要跟踪对象变化时降低内存消耗。

```python
#hello.py 配置mysql数据库
from flask_sqlalchemy import SQLAlchemy
import pymysql

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI']='mysql+pymysql://username:password@hostname/databsae'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS']=False
db = SQLAlchemy(app)
```

## 定义模型

模型这个术语表示应用使用的持久化实体。在ORM中，模型一般是一个Python类，类中的属性对应于数据库表中的列。

```python
class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.Integer,primary_key=True)
    name = db.Column(db.String(64),unique=True)
    
    def __repr__(self):
        return '<Role %r>' % self.name

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer,primary_key=True)
    username = db.Column(db.String(64),unique=True)
    
    def __repr__(self):
        return '<User %r>' % self.username
```

类变量`__tablename__`定义在数据库中使用的表名。如果没有定义`__tablename__`，Flask-SQLAlchemy会使用一个默认名称。但默认的表名没有遵守流行的复数命名的约定。其余变量都是该模型的属性，定义为`db.Column`类的实例。虽然没有强制要求，但这两个模型都定义了`__repr()__`方法，返回一个具有可读性的字符串表示模型，供调试和测试时使用。

**最常用的SQLAlchemy列类型**

| **类型名**   | **python**中类型  | **说明**                                            |
| ------------ | ----------------- | --------------------------------------------------- |
| Integer      | int               | 普通整数，一般是32位                                |
| SmallInteger | int               | 取值范围小的整数，一般是16位                        |
| BigInteger   | int或long         | 不限制精度的整数                                    |
| Float        | float             | 浮点数                                              |
| Numeric      | decimal.Decimal   | 普通整数，一般是32位                                |
| String       | str               | 变长字符串                                          |
| Text         | str               | 变长字符串，对较长或不限长度的字符串做了优化        |
| Unicode      | unicode           | 变长Unicode字符串                                   |
| UnicodeText  | unicode           | 变长Unicode字符串，对较长或不限长度的字符串做了优化 |
| Boolean      | bool              | 布尔值                                              |
| Date         | datetime.date     | 时间                                                |
| Time         | datetime.datetime | 日期和时间                                          |
| LargeBinary  | str               | 二进制文件                                          |

**最常用的SQLAlchemy列选项**

| 选项名      | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| primary_key | 如果设为True，列为表的主键                                   |
| unique      | 如果设为True，列不允许出现重复的值                           |
| index       | 如果设为True，为列创建索引，提升查询效率                     |
| nullable    | 如果设为True，列允许使用空值；如果设为False，列不允许使用空值 |
| default     | 为列定义默认值                                               |

## 关系

关系型数据库使用关系把不同的行联系起来。一对多关系在模型类中的表示方法如下。

```python
#hello.py 在数据库模型中定义关系
class Role(db.Model:
    #...
    users = db.relationship('User',backref='role')
 
class User(db.Model:
    #...
    role_id = db.Column(db.Integer,db.ForeignKey('roles.id'))
```

关系使用users表中的外键连接两行，添加到User模型中的role_id列被定义为外键，就是这个外键建立起了关系。

从“一”那一端可见，添加到Role模型中的users属性代表这个关系的面向对象视角。对于一个Role类的实例，其users属性将返回与角色相关联的用户组成的列表。`db.relationship()`第一个参数表明这个关系的另一端是哪个模型。其中的`backref`参数向User模型中添加一个role属性，从而定义反向关系。通过User实例的这个属性可以获取对应的Role模型对象，而不用通过role_id外键获取。

**常用的SQLAlchemy关系选项**

| 选项名        | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| backref       | 在关系的另一个模型中添加反向引用                             |
| primaryjoin   | 明确指定两个模型之间使用的联结条件，只在模棱两可的关系中需要指定 |
| lazy          | 指定如何加载相关记录，可选值有select（首次访问时按需加载）、immediate（源对象加载后就加载）、joined（加载记录，但使用联结）、subquery（立即加载，但使用子查询）、noload（永不加载）和dynamic（不加载记录，但提供加载记录的查询） |
| uselist       | 如果设为False，不使用列表，而使用标量值                      |
| order_by      | 指定关系中记录的排序方式                                     |
| secondary     | 指定多对多关系中关联表的名称                                 |
| secondaryjoin | SQLAlchemy无法自行决定时，指定多对多关系中的二级联结条件     |

除了一对多之外，还有其他几种关系类型。**一对一**关系可以用前面介绍的一对多关系表示，但调用`db.relationship()`时要把`uselist`设为False，把“多”变成“一”。**多对一**关系也可以使用一对多表示，对调两个表即可，或者把外键和`db.relationship()`都放在“多“这一侧。关系最复杂的是**多对多**，需要用到第三张表，这个表成为**关联表**或**联结表**。

## 数据库操作

### 创建表

首先，要让Flask-SQLAlchemy根据模型来创建数据库。`db.create_all()`函数将寻找所有db.Model的子类，然后在数据库中创建对应的表。

```
(venv) $ flask shell
>>> from hello import db
>>> db.create_all()
```

如果数据库表已经存在于数据库中，那么`db.create_all()`不会重新创建或更新相应的表，如果修改模型后要把改动应用到现有的数据库中，这一行为会带来不便，更新现有的数据库表的蛮力方式是先删除旧表再重新创建。

```shell
>>> db.drop_all()
>>> db.create_all()
```

当然这种方式会将旧表中的所有数据都销毁了。这不是我们想看到的。后面将说明更好的数据库更新方式。

### 插入行

```python
from hello import Role,User
admin_role = Role(name='Admin')
user_john = User(username='john',role=admin)
db.session.add(admin)
db.session.add(user_john)
#或者db.session.add_all([admin,user_john])
db.session.commit()
```

数据库会话能保证数据库的一致性，提交操作使用原子方式把会话中的对象全部写入数据库，如果在写入会话的过程中发生了错误，那么整个会话都会失效。数据库会话也可以**回滚**。调用`db.session.rollback()`后，添加到数据库会话中的所有对象都将还原到他们在数据库中的状态。

### 修改行

```python
admin_role.name = 'Administrator'
db.session.add(admin_role)
db.session.commit()
```

### 删除行

```shell
db.session.delete(admin_role)
db.session.commit()
```

### 查询行

Flask-SQLAlchemy为每个模型类都提供了query对象。最基本的模型查询是使用all()方法取回对应表中的所有记录。

```shell
>>> Role.query.all()
[<Role 'Administrator'>]
```

使用过滤器可以配置query对象进行更精确的的数据库查询

```shell
>>> User.query.filter_by(role=admin_role).all()
[<User 'john'>]
```

如果推出了shell会话，前面的这些例子中创建的对象就不会以Python对象的形式存在，但在数据库中仍有对应的行，如果打开一个新的shell会话，就要从数据库中读取行。重新创建Python对象。

```shell
>>> admin_role = Role.query.filter_by(name='Administrator').first()
```

**常用的SQLAlchemy查询过滤器**

| 过滤器      | 返回结果                                         |
| ----------- | ------------------------------------------------ |
| filter()    | 把过滤器添加到原查询上，返回一个新查询           |
| filter_by() | 把等值过滤器添加到原查询上，返回一个新查询       |
| limit       | 使用指定的值限定原查询返回的结果                 |
| offset()    | 偏移原查询返回的结果，返回一个新查询             |
| order_by()  | 根据指定条件对原查询结果进行排序，返回一个新查询 |
| group_by()  | 根据指定条件对原查询结果进行分组，返回一个新查询 |

**最常用的SQLAlchemy查询执行方法**

| 执行函数       | 返回结果                                     |
| -------------- | -------------------------------------------- |
| all()          | 以列表形式返回查询的所有结果                 |
| first()        | 返回查询的第一个结果，如果未查到，返回None   |
| first_or_404() | 返回查询的第一个结果，如果未查到，返回404    |
| get()          | 返回指定主键对应的行，如不存在，返回None     |
| get_or_404()   | 返回指定主键对应的行，如不存在，返回404      |
| count()        | 返回查询结果的数量                           |
| paginate()     | 返回一个Paginate对象，它包含指定范围内的结果 |

## 在视图函数中操作数据库

```python
#hello.py 在视图函数中操作数据库
@app.route('/',methods=['GET','POST'])
def index():
	form = NameForm()
	if form.validate_on_submit():
		user = User.query.filter_by(username=form.name.data).first()
		if user is None:
			user = User(username=form.name.data)
			db.session.add(user)
			db.session.commit()
			session['known'] = False
		else:
			session['known'] = True
		session['name']=form.name.data
        form.name.data=''
        return redirect(url_for('index'))
    return render_template('index.html',form=form,name=session.get('name'),
                           know=session.get('know',False))
```

```jinja2
#templates/index.html 在模板中定制欢迎消息
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}
{% block titile %}Flasky{% endblock %}

{% block page_content %}
<div class="page_content">
	<h1>hello,{% if name %}{{ name }}{% else %}Stranger{% endif %}!</h1>
	{% if not known %}
	<p>Pleased to meet you!</p>
	{% else %}
	<p>Happy to see you again!</p>
	{% endif %}
</div>
{{ wtf.quick_form(form) }}
{% endblock %}
```

## 集成Python shell

每次启动shell会话都要导入数据库实例和模型，为了避免一直重复导入，我们可以做些配置，让flask shell命令自动导入这些对象。

若想把对象添加导入列表中，必须使用app.shell_context_processor装饰器创建并注册一个**shell上下文处理器**

```python
@app.shell_context_processor
def make_shell_context():
    return dict(db=db,User=User,Role=Role)
```

## 使用Flask-Migrate实现数据库迁移

在开发应用的过程中，你会发现有时候需要修改数据库模型，修改之后还要更新数据库。按照传统做法必须先删除数据库表，然后重新创建数据库表，这样做会导致丢失数据库全部数据，更新表最好的方法是使用**数据库迁移**框架。SQLAlchemy的开发人员编写了一个迁移框架，名为Alembic。除了这个之外还可以使用Flask-Migrate扩展。

### 创建迁移仓库

首先安装Flask-Migrate：

```shell
(venv) $ pip install flask-migrate
```

```python
#hello.py 初始化Flask-Migrate
from flask_migrate import Migrate
#...

migrate = Migrate(app,db)
```

为了开放数据库迁移相关的命令，Flask-Migrate添加了flask db命令和几个子命令。再新项目中可以使用init子命令添加数据库迁移支持：

```shell
(venv) $ flask db init
```

### 创建迁移脚本

使用Flask-Migrate管理数据库模式变化的步骤如下：

1. 对模型类做必要的修改。
2. 执行`flask db migrate `命令，自动创建一个迁移脚本
3. 检查自动生成的脚本，根据对模型的实际改动进行调整
4. 把迁移脚本纳入版本控制
5. 执行`flask db upgrade`命令，把迁移应用到数据库中

### 添加几个迁移

在开发项目的过程中，时常需要修改数据库模型。

1. 对模型类做必要的修改。
2. 执行`flask db migrate `命令，自动创建一个迁移脚本
3. 检查自动生成的脚本，根据对模型的实际改动进行调整
4. 执行`flask db upgrade`命令，把迁移应用到数据库中

实现一个功能时，可能需要多次修改数据库模型才能得到预期结果，如果前一个迁移脚本还未提交到源码控制系统中，还可以继续在那个迁移中修改。

1. 执行`flask db downgrade`命令，还原前一个脚本对数据库的改动（注意，这可能导致部分数据丢失）
2. 删除前一个迁移脚本，因为现在已经没什么用了
3. 执行`flask db migrate`命令生成另一个新的数据库迁移脚本
4. 根据前面的说明，检查并应用迁移脚本