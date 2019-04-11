title: 理解Flasky应用用户角色管理
author: 追梦人

toc: true

tags:

  - python
  - Flask
  - UserPermission
categories: []
date: 2019-04-09 13:42:00

---

# 用户角色

Web应用中的用户并非都具有同等地位。在多数应用中，一小部分可信用户具有额外权限，用于保障应用平稳运行。管理员就是最好的例子，但有时也需要介于管理员和普通用户之间的角色，例如内同协管员。为此，要为所有用户分配一个角色。

本文介绍的用户角色实现方式结合了分立的角色和权限，赋予用户分立的角色，但是各个角色通过权限列表定义允许用户执行的操作。

<!-- more -->

## 用户在数据库中的表示

```python
#app/models.py 角色数据库模型
class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.Integer,primary_key = True)
    name = db.Column(db.String(64),unique=True)
    default = db.Column(db.Boolean,default=False,index=True)
    permissions = db.Column(db.Integer)
    users = db.relationship('User',backref='role',lazy='dynamic')
    
    def __init__(self,**kwargs):
        super(Role,self).__init__(**kwargs)
        if self.permissions is None:
            self.permissions = 0
```

这个模型新增了`default`字段，只能有一个角色的这个字段可以设为True，其他角色都应该设为False。另一处改动是添加了`permissions`字段，其值是一个整数，以简洁的方式定义一组权限。`SQLAlchemy`默认把这个字段的值设为None，因此我们添加了一个类构造函数，在为给构造函数提供参数的时候，把这个字段设置为0。

**应用中的各项权限**

| 操作                   | 权限名   | 权限值 |
| ---------------------- | -------- | ------ |
| 关注用户               | FOLLOW   | 1      |
| 在他人的文章中发表评论 | COMMENT  | 2      |
| 写文章                 | WRITE    | 4      |
| 管理他人发表的评论     | MODERATE | 8      |
| 管理员权限             | ADMIN    | 16     |

**使用2的幂表示权限值有个好处：每种各不同的权限组合对应的值都是唯一的，方便存入角色的`permissions`字段。**例如，若想为一个用户角色赋予权限，使其能够关注其他用户，并且在文章中发表评论，则权限值为`FOLLOW + COMMENT = 3`。通过这种方式存储各个角色的权限特别高效。

```python
#app/models.py 权限常量
class Permission:
    FOLLOW = 1
    COMMENT = 2
    WRITE = 4
    MODERATE = 8
    ADMIN = 16
```

添加这些权限向量之后，可以在Role模型中定义几个新方法，用于管理权限。

```python
#app/models.py Role模型中管理权限的方法
class Role(db.Model):
    #...
    def add_permission(self,perm):
        if not self.has_permission(perm):
            self.permissions += perm
    
    def remove_permission(self,perm)
    	if self.has_permission(perm):
            self.permission -= perm
      
   	def reset_permissions(self):
        self.permission = 0
       
    def has_permission(self,perm):
        return self.permission & perm == perm
```

`add_permission、remove_permisson、reset_permissions`这三个方法使用基本的算法运算符更新权限列表。`has_permission`方式几种方法中最复杂的，**它使用位与运算符&检查组合权限是否包含指定的单独权限。**

**用户角色**

| 用户角色 | 权限                                    | 说明                                                         |
| -------- | --------------------------------------- | ------------------------------------------------------------ |
| 匿名     | 无                                      | 对应只读权限，这是未登录的未知用户                           |
| 用户     | FOLLOW、COMMENT、WRITE                  | 具有发布文章、发表评论和关注其他用户的权限；这是新用户的默认角色 |
| 协管员   | FOLLOW、COMMENT、WRITE、MODERATE        | 增加管理其他用户所发表评论的权限                             |
| 管理员   | FOLLOW、COMMENT、WRITE、MODERATE、ADMIN | 具有所有权限，包括修改其他用户所属角色的权限                 |

将角色手动添加到数据库中既耗时又容易出错。作为替代，我们可以在Role类中添加一个类方法，完成这个操作。

```python
#app/models.py 在数据库中创建角色
class Role(db.Model):
    #...
    @staticmethod
    def insert_roles():
        roles = {
            'User':[Permisson.FOLLOW,Permission.COMMENT,Permission.WRITE],
            'Moderator':[Permisson.FOLLOW,Permission.COMMENT,Permission.WRITE,
                         Permission.MODERATE],
            'Administrator':[Permisson.FOLLOW,Permission.COMMENT,Permission.WRITE,
                         Permission.MODERATE,Permission.ADMIN]
        }
        default_role = 'User'
        for r in roles:
            role = Role.query.filter_by(name=r).first()
            if role is None:
                role = Role(name=r)
            role.reset_permissions()
            for perm in roles[r]:
                role.add_permission(perm)
            role.default = (role.name == default_role )
            db.session.add(role)
        db.session.commit()
```

`insert_roles()`函数并不是直接创建新角色对象，而是通过角色名查找现有的角色，然后再进行更新。注意，“匿名”角色不需要在数据库中表示出来。

此外还要注意，`insert_roles()`是**静态方法**。这是一种特殊的方法，无须创建对象，而是直接在类上调用，例如`Role.insert_roles()`。

## 赋予角色

用户在应用中注册账户时，应该赋予其适当的角色。多数用户在注册时赋予的角色时“用户”，这是默认角色。唯一例外的是管理员，管理员在最开始就应该赋予“管理员”角色。管理员保存在设置变量`FLASKY_ADMIN`中的电子邮件地址识别，只要这个电子邮件地址出现在注册请求中，就会被赋予正确的角色。

```python
#app/models.py 定义默认的用户角色
class User(UseMixin,db.Model):
    #...
    def __init__(self,**kwargs):
        super(User,self).__init__(**kwargs)
        if self.role is None:
            if self.email == current_app.config['FLASKY_ADMIN']
            	self.role = Role.query.filter_by(name='Administrator').first()
            if self.role is None:
                self.role = Role.query.filter_by(default = True).first()
    #...
```

## 检验角色

为了简化角色和权限的实现过程，可在User模型中添加一个辅助方法，检查赋予用户的角色是否有某项权限。这个辅助方法的实现很简单，直接委托前面添加的权限管理方法。

```python
#app/models.py 检查用户是否有指定的权限
class User(UserMixin,db.Model):
    #...
    
    def can(self,perm):
        return self.role is not None and self.role.has_permisson(perm)
   	def is_administrator(self):
        return self.can(Permission.ADMIN)
   
class AnonymousUser(AnonymousUserMixin):
    def can(self,permissions):
        return False
    def is_administrator(self):
        return False
   
login_manager.anonymous_user = AnonymousUser
```

如果角色中包含请求的权限，那么User模型中添加的`can()`方法会返回True，表示允许用户执行此项操作。因为经常需要检查是否具有管理员权限，所有还单独实现了`is_administrator()`方法。

为了方便操作，我们还定义了`AnonymousUser`类，并实现了`can()`方法和`is_administrator()`方法。这样应用无须检查用户是否登陆，就能放心调用`current_user.can()`和`current_user.is_administrator()`。我们通过`login_manager.anonymou_user`属性告诉`Flask-Login`使用应用自定义的匿名用户类。

**如果想让视图函数只对具有特定权限的用户开放，可以使用自定义的装饰器。下面示例中实现了两个装饰器，一个用于检查常规权限，另一个专门检查管理员权限。**

```python
#app/decorators.py 检查用户权限的自定义装饰器
from functools import wraps
from flask import abort
from flask_login import current_user
from .models import Permission

def permission_required(permission):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if not current_user.can(permission):
                abort(403)
            return f(*args, **kwargs)
        return decorated_function
    return decorator

def admin_required(f):
    return permission_required(Permission.ADMIN)(f)
```

这两个装饰器都使用了**Python标准库中的functools包**(https://docs.python.org/3/library/functools.html)，如果用户不具有指定权限，则返回403错误响应。

在模板中可能也需要检查权限，所以Permission类中的所有变量要能在模板中访问。为了避免每次调用`render_template()`时都多添加一个模板参数，可以使用**上下文处理器**。在渲染时，上下文处理器能让变量在所有模板中可访问。

```python
#app/main/__init__.py 把Permission类加入模板上下文
@main.app_context_processor
def inject_permissions():
    return dict(Permission = Permission)
```

