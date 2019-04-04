title: 理解Flask应用用户身份验证
author: 追梦人

toc: true

tags:

  - python
  - Flask
  - Flask-Login
categories: []
date: 2019-04-02 12:12:00

---

# 用户身份验证

多数应用都要记录用户是谁。用户连接应用时验证身份，通过这一过程，让应用知道自己是的身份。应用知道用户是谁后，就能提供有针对性的体验。

最常见的身份验证方法要求用户提供一个身份证明，可以是用户的电子邮件地址，也可以是用户名，以及一个只有用户自己知道的密令。

<!-- more -->

## Flask的身份验证扩展

优秀的Python身份验证包很多，但没有一个能实现所有功能。本文介绍的身份验证方案将使用多个包，而且还要编写胶水代码，让不同的包良好协作。

- Flask-Login：管理已登陆用户的用户会话
- Werkzeug：计算密码散列值并进行核对
- itsdangerous：生成并核对加密安全令牌

## 密码安全性

设计Web应用时，人们往往会忽视数据库中用户信息的安全性，如果攻击者入侵服务器，攥取了数据库，用户的安全就处在风险之中，而且这个风险超乎你的想象。总所周知，多数用户会在不同的网站使用相同的密码。因此，即便不保存任何敏感信息，攻击者获得存储在数据库中的密码之后，也能访问用户在其他网站中的账户。

若想保证数据库中用户密码的安全，关键在于不存储密码本身，而是存储密码的**散列值**，计算密码散列值的函数接收密码作为输入，添加随机内容（**盐值**）之后，使用多种单向加密算法转换密码，最终得到一个和原始密码没有关系的字符序列，而且无法还原成原始密码。

## 使用Werkzeug计算密码散列值

```shell
(venv) $ pip install werkzeug
```

Werkzeug中的security模块实现了密码散列值的计算。这一功能的实现只需要两个函数，分别用在注册和核对两个阶段。

```python
generate_password_hash(password,method='pbkdf2:sha256',salt_length=8)
```

这个函数的输入为原始密码，返回密码散列值的字符串形式，供存入用户数据库。`method`和`salt_length`的默认值就能满足大多数需求。

```
check_password_hash(hash,password)
```

这个函数的参数是从数据库中取回的密码散列值和用户输入的密码。返回值为True时表明用户输入的密码正确。

```python
#app/models.py 在User模型中加入密码散列
from werkzeug.security import generate_password_hash,check_password_hash

class User(db.Model):
    #...
    password_hash = db.Column(db.String(128))
    
    @property
    def password(self):
        raise AttributeError('password is not a readable attribute')
        
    @password.setter
    def password(self,password):
        self.password_hash = generate_password_hash(password)
    def verify_password(self,password)
    	return check_password_hash(self.password_hash,password)
```

计算密码散列值的函数通过名为password的只写属性实现。设定这个属性的值时，赋值方法会调用Wekzeug提供的`generate_password_hash()`函数，并把得到的结果写入`password_hash`字段。如果视图读取password属性的值，则会返回错误，原因很明显，因为生成散列值后就无法还原成原来的密码了。

`verify_password()`方法接受一个参数（即密码），将其传给Werkzeug提供的`check_password_hash()`。

## 创建身份验证蓝本

本节将在一个新蓝本中定义与用户身份验证子系统相关的路由，这个蓝本名为auth。把应用的不同子系统放在不同的蓝本中，有利于保持代码的整洁有序。添加auth子目录。

```python
#多文件Flask应用的基础结构
|-flasky
  |-app/
  	|-templates/
  	|-static/
    |-auth/ 	#新添加的auth子目录
  	  |-__init__.py
  	  |-forms.py
  	  |-views.py
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

```python
#app/auth/__init__.py 创建身份验证蓝本
from flask import Blueprint

auth = Blueprint('auth',__name__)

from . import views
```

```python
#app/auth/views.py 身份验证蓝本中的路由和视图函数
from flask import render_template
from . import auth

@auth.route('/login')
def login():
    return render_template('auth/login.html')
```

这段代码添加了一个/login路由，渲染同名的占位模板。注意，为`render_template()`指定的模板文件保存在auth目录中。这个目录必须在app/templates中创建。

```python
#app/__init__.py 注册身份验证蓝本
def create_app(config_name):
    #...
    from .auth import auth as auth_blueprint
    app.register_blueprint(auth_blueprint,url_prefix='/auth')
    
    return app
```

注册蓝本时使用的`url_prefix`是可选参数。如果使用了这个参数，注册后蓝本中定义的所有路由都会加上指定的前缀，即这个例子中的`/auth`。例如，/login路由会注册成为`/auth/login`，在开发Web服务器中，完整的URL就变成了`http://localhsot:5000/auth/login`。

## 使用Flask-Login验证用户身份

用户登陆应用后，他们的验证状态要记录在用户会话中，这样浏览不同的页面时才能记住这个状态。Flask-Login是一个非常有用的小型扩展，专门用于管理身份验证系统中的验证状态，且不依赖特定的身份验证机制。

```shell
(venv) $ pip install flask-login
```

### 准备用于登陆的用户模型

Flask-Login的运转需要应用中的User对象。要想使用Flask-Login扩展，应用的User模型必须实现几个属性和方法。

**Flask-Login要求实现的属性和方法**

| 属性/方法        | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| is_authenticated | 如果用户提供的登陆凭据有效，必须返回True，否则返回False      |
| is_active        | 如果允许用户登陆，必须返回True，否则返回False。如果想禁用账户，可以返回False |
| is_anonymous     | 对普通用户必须始终返回False，如果是表示匿名用户的特殊用户对象，应该返回True |
| get_id()         | 必须返回用户的唯一标识符，使用Unicode编码字符串              |

这些属性和方法可以直接在模型类中实现，不过还有一种更简单的替代方案。Flask-Login提供了一个UserMixin类，其中包含默认实现，能满足多数需求。修改后的User模型如下所示。

```python
#app/models.py 修改User模型，支持用户登陆
from flask_login import UserMixin

class User(UserMixin,db.Model):
    __tablename__='users'
    id = db.Column(db.Integer,primary_key = True)
    email = db.Column(db.String(64),unique=True,index=True)
    username = db.Column(db.String(64),unique=True,index=True)
    password_hash = db.Column(db.String(128))
    role_id = db.Column(db.Integer,db.ForeignKey('roles.id'))
```

**Flask-Login在应用的工厂函数中初始化**

```python
#app/__init__.py 初始化Flask-Login
from flask_login import LoginManager

login_manager = LoginManager()
login_manager.login_view = 'auth.login'

def create_app(config_name):
    #...
    login_manager.init_app(app)
    #...
```

`LoginManager`对象`login_view`属性用于设置登陆页面的端点。匿名用户尝试访问受保护的页面时，`Flask-Login`将重定向到登陆页面。因为登陆路由在蓝本中定义，所以要在前面加上蓝本的名称。

最后，`Flask-Login`要求应用指定一个函数，在扩展需要从数据库中获取指定标识符对应的用户时调用。

```python
#app/models.py 加载用户的函数
from . import login_manager

@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))
```

`login_manager.user_loader`装饰器把这个函数注册给`Flask-Login`，在这个扩展需要获取已登陆用户的信息时调用。传入的用户标识符是个字符串，因此这个函数先把标识符转换成整数，然后传给`Flask-SQLAlchemy`查询，加载用户。

### 保护路由

为了保护路由，只让通过身份验证的用户访问，`Flask-Login`提供了一个`login_required`装饰器。其用法演示如下：

```python
from flask_login import login_required

@app.route('/secret')
@login_required
def secret():
    return 'Only authenticated users are allowed'
```

从这个实例可以看初，多个函数装饰器可以叠加使用，函数上有多个装饰器时，各装饰器只对随后的装饰器和目标函数起作用。在这个示例中，`secret()`函数受`login_required`装饰器的保护，禁止未授权的用户访问。得到的函数又注册为一个Flask路由。如果调换两个装饰器，得到的结果将是错的，因为原始函数先注册为路由，然后才从`login_required`装饰器接收到额外的属性。

### 添加登陆表单

```python
#app/auth/forms.py 登陆表单
from flask_wtf import FlaskForm
from wtforms import StringField,PasswordField,BooleanField,SubmitField
from wtforms.validators import DataRequired,length,Email

class LoginForm(FlaskForm):
    email = StringField('电子邮箱',validators=[DataRequired(),length(1,64),Email()])
    password = PasswordField('密码',validators=[DataRequired()])
    remember_me = BooleanField('记住我')
    submit = SubmitField('登陆')
```

### 登入用户

```python
#app/auth/views.py 登陆路由
from flask import render_template,redirect,request,url_for,flash
from flask_login import login_user
from . import auth
from ..models import User
from .forms import LoginForm

@auth.route('/login',methods=['GET','POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first()
        if user is not None and user.verify_password(form.password.data):
            login_user(user,form.remenber_me.data)
            next = request.args.get('next')
            if next is None or not next.startswith('/'):
                next = url_for('main.index')
            return redirect(next)
        flash('用户名或密码无效')
     return render_template('auth/login.html',form=form)
```

如果密码正确，调用`Flask-Login`的`login_user()`函数，在用户会话中把用户标记为已登陆。`login_user()`函数的参数是要登陆的用户，以及可选的“记住我”布尔值，如果这个字段的值为False，关闭浏览器后用户会话就过期了，所以下次需要重新登陆。如果值为True，那么会在用户浏览器中写入一个长期有效的cookie，使用这个cookie可以复现用户会话。cookie默认记住一年，可以使用可选的`REMENBER_COOKIE_DURATION`配置选项更改这个值。

用户访问未授权的URL时会显示登陆表单，`Flask-Login`会把原URL保存在查询字符串的next参数，则重定向到首页。next参数中的URL会经验证，确保是相对URL，Python `startswith()` 方法用于检查字符串是否是以指定子字符串开头，如果是则返回 True，否则返回 False。以防恶意用户利用这个参数，把不知情的用户重定向到其他网站。

### 登出用户

```python
from flask_login import logout_user,login_required

@auth.route('/logout')
@login_required
def logout():
    logout_user()
    flash('您已退出登陆')
    return redirect(url_for('main.index'))
```

## 注册新用户

如果新用户想要成为应用的成员，必须在应用中注册。

### 添加用户注册表单

```python
#app/auth/forms.py 用户注册表单
from flask_wtf import FlaskForm
from wtforms import StringField,PasswordField,BooleanField,SubmitField
from wtforms.validators import DataRequired,length,Email,Regexp,EqualTo
from wtforms import ValidationError
from ..models import User

class RegistrationForm(FlaskForm):
    email = StringField('电子邮箱',validators=[DataRequired(),length(1,64),Email()])
    username = StringField('用户名',validators=[DataRequired(),length(1,64),  
        		Regexp('^[A-Za-z][A-Za-z0-9_]*$',0,
                       '用户名必须仅为多为字母，数字，或者下划线组成！')])
    password = PasswordField('密码',validators=[DataRequired(),
                 EqualTo('password2',message='两次密码必须一样！')])
    password2= PasswordField('确认密码',validators=[DataRequired()])
    submit = SubmitField('注册')

    def validate_email(self,field):
        if User.query.filter_by(email=field.data).first():
            raise ValidationError('该邮箱已被注册，请重新输入！')

    def validate_username(self,field):
        if User.query.filter_by(username=field.data).first():
            raise ValidationError('该用户名已被注册，请重新输入！')
```

这个表单还有**两个自定义的验证函数**，以方法的形式实现。如果表单类定义了以`validate_`开头且后面跟着字段名的方法，这个方法就和常规的验证函数一起调用。本例分别为email和username字段定义了验证函数，确保填写的值在数据库中没出现过。自定义的验证函数想要表示验证失败，可以抛出`ValidationError`异常。

### 注册新用户

```python
#app/auth/views.py 用户注册路由
@auth.route('/register',methods=['GET','POST'])
def register():
    form = RegistrationForm()
    if form.validate_on_submit():
        user = User(email=form.email.data,
                    username=form.username.data,
                    password=form.password.data)
        db.session.add(user)
        db.session.commit()
        return redirect(url_for('auth.login'))
    return render_template('auth/register.html',form=form)
```

## 确认账户

对于某些特定类型的应用，有必要确认注册时用户提供的信息是否正确。为了确认电子邮件地址，用户注册后，应用会立即发送一封确认邮件。新账户先被标记成待确认状态，用户按照邮件中的说明操作后，才能证明自己可以收到电子邮件，账户确认过程中，往往会要求用户点击一个包含确认令牌的特殊URL链接。

### 使用itsdangerous生成确认令牌

确认邮件中最简单的确认链接是`http://www.example.com/auth/confirm/<id>`这种形式的URL，其中<id>是数据库分配给用户的数字id。用户点击链接后，处理这种路由的视图函数将确认收到的用户id，然后将用户状态更新为已确认。

但这种实现方式显然不是很安全，只要用户能判断确认链接的格式，就可以随便指定URL中的数字，从而确认任意账户。解决方法是把URL中的<id>换成包含相同信息的令牌，但是只有服务器才能生成有效的确认URL。

Flask使用加密的签名cookie保持用户会话，以防止被篡改。用户会话cookie中有一个由`itsdangerous`包生成的加密签名。如果用户会话内容被篡改，签名将不再与内容匹配，这样会使Flask销毁会话，然后重建一个。

```shell
(venv) $ flask shell
>>> from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
>>> s = Serializer(app.config['SECRET_KEY'],expires_in=3600)
>>> token = s.dumps({'confirm':23})
>>> token
b'eyJhbGciOiJIUzUxMiIsImlhdCI6MTU1NDE4OTgxMiwiZXhwIjoxNTU0MTkzNDEyfQ.eyJjb25maXJtIjoyM30.8tsXoL9ar221F_MW5MRR7ifGE0J3duL_BF_0YZfbfIPjVks76hAjWa02CKZqZ177EvZp4VW4Hx5EzVwesWxydA'
>>> data = s.loads(token)
>>> data
{'confirm': 23}
```

`itsdangerous`提供了多种生成令牌的方法。其中，`TimedJSONWebSignatureSerializer`类生成具有过期时间的`JSON Web`签名（JWS）。这个类的构造函数接受参数是一个密钥。

`dumps()`方法为指定的数据生成一个加密签名，然后再对数据和签名进行序列化，生成令牌字符串。`expires_in`参数设置令牌过期时间，单位为秒。

为了解码令牌，序列化对象提供了`loads()`方法，其唯一参数是令牌的字符串。这个方法会检验签名和过期时间，如果都有效，则返回原始数据。

```python
#app/models.py 确认用户账户
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
from flask import current_app
from . import db

class User(UserMixin,db.Model):
    #...
    confirmed = db.Column(db.Boolean,default=False)
    
    def generate_confirmation_token(self,expiration=3600):
        s = Serializer(current_app.config['SECRET_KEY'],expiration)
        return s.dumps({'confirm':self.id}).decode('utf-8')
    
    def confirm(self,token):
        s = Serializer(current_app.config['SECRET_KEY'])
        try:
            data = s.loads(token.encode('utf-8'))
        except:
            return False
        if data.get('confirm') != self.id:
            return False
        self.confirmed = True
        db.session.add(self)
        return True
```

### 发送确认邮件

```python
#app/auth/views.py 能发送确认邮件的注册路由
from ..email import send_email

@auth.route('/register',methods=['GET','POST'])
def register():
    form = RegistrationForm()
    if form.validate_on_submit():
        user = User(email=form.email.data,
                    username=form.username.data,
                    password=form.password.data)
        db.session.add(user)
        db.session.commit()
        token = user.generate_confirmation_token()
        send_email(user.email,'请确认您的账户','auth/email/confirm',user=user,token=token)
        flash('确认账户邮件已发送到您的邮箱，请登陆您的电子邮箱确认！')
        return redirect(url_for('auth.login'))
    return render_template('auth/register.html',form=form)
```

**确认账户的视图函数**

```python
#app/auth/views.py 确认用户的账户
from flask_login import current_user

@auth.route('/confirm/<token>')
@login_required
def confirm(token):
    if current_user.confirmed:
        return redirect(url_for('main.index'))
    if current_user.confirm(token):
        db.session.commit()
        flash('您已经确认了您的账户，谢谢！')
    else:
        flash('这个确认链接无效或者已经失效！')
    return redirect(url_for('main.index'))
```

`Flask-Login`提供的`login_required`装饰器会保护这个路由，因此，用户点击确认邮件中的链接后，要先登陆，才能执行这个视图函数。

各个应用可以自行决定用户确认账户之前可以做哪些操作。比如，允许为未确认的用户登陆，但只显示一个页面，要求用户在获取进一步访问权限之前先确认账户。

这一步可使用Flask提供的`before_request`钩子完成，对于蓝本来说，`before_request`钩子只能应用到属于蓝本的请求上，若想在蓝本中使用针对应用全局请求的钩子，必须使用`before_app_request`装饰器。

```python
@auth.before_app_request
def before_request():
    if current_user.is_authenticated and not current_user.confirmed \
        and request.blueprint !='auth' and request.endpoint !='static':
        return redirect(url_for('auth.unconfirmed'))

@auth.route('/unconfirmed')
def unconfirmed():
    if current_user.is_anonymous or current_user.confirmed:
        return redirect(url_for('main.index'))
    return render_template('auth/unconfirmed.html')
```

同时满足以下3个条件时，before_app_request处理程序会拦截请求

- 用户已登陆(current_user.is_authenticated == True)
- 用户的账户还未确认
- 请求的URL不在身份验证蓝本中，而且也不是对静态文件的请求

如果请求满足以上条件，会被重定向到`/auth/unconfirmed`路由，显示一个确认账户相关信息的页面。

此外还有一个链接，用于请求发送新的确认邮件，以防以前的邮件丢失。重新发送确认邮件的路由。

```python
@auth.route('/confirm')
@login_required
def resend_confirmation():
    token = current_user.generate_confirmation_token()
    send_email(current_user.email, '请确认您的账户', 
               'auth/email/confirm', user=current_user, token=token)
    flash('确认账户邮件已发送到您的邮箱，请登陆您的电子邮箱确认！')
    return redirect(url_for('main.index'))
```

## 管理账户

拥有应用账户的用户有时可能需要修改账户信息。

### 修改密码

**修改密码表单**

```python
#auth/forms.py 修改密码表单

class ChangePasswordForm(FlaskForm):
    old_password =PasswordField('旧密码',validators=[DataRequired()])
    password = PasswordField('新密码',validators=[DataRequired(), 
                  EqualTo('password2',message='重新输入的两次密码必须一致')])
    password2= PasswordField('确认新密码',validators=[DataRequired()])
    submit = SubmitField('更改密码')

```

**修改密码路由**

```python
#auth/views.py 修改密码路由

@auth.route('change-password',methods=['GET','POST'])
@login_required
def change_password():
    form = ChangePasswordForm()
    if form.validate_on_submit():
        if current_user.verify_password(form.old_password.data):
            current_user.password = form.password.data
            db.session.add(current_user)
            db.session.commit()
            flash('您的密码已更改！')
            return redirect(url_for('main.index'))
        else:
            flash('密码错误！请重新输入')
    return render_template('auth/change_password.html',form=form)
```

### 重置密码

有时忘记密码，需要重置密码

**忘记密码表单**

```python
#auth/forms.py 忘记密码，重设密码

class PasswordResetRequestForm(FlaskForm):
    email = StringField('请输入您的电子邮箱',validators=[DataRequired(),
                                           length(1,64),Email()])
    submit = SubmitField('发送邮件')
```

**重置密码路由**

```python
#auth/views.py 忘记密码，重置密码
@auth.route('reset',methods=['GET','POST'])
def password_reset_request():
    if not current_user.is_anonymous:
        return redirect(url_for('main.index'))
    form = PasswordResetRequestForm()
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first()
        if user:
            token = user.generate_reset_token()
            send_email(user.email,'重置您的密码',
                       'auth/email/reset_password',user=user,token=token)
            flash('我们已向您发送一封电子邮件，其中包含重置密码的说明')
        else:
            flash('无效的电子邮箱，请重新输入！')
        return redirect(url_for('auth.login'))
    return render_template('auth/reset_password.html',form=form)
```

**产生重置密码的token**

```python
#app/models.py 产生重置密码token
def generate_reset_token(self,expiration=3600):
    s = Serializer(current_app.config['SECRET_KEY'],expiration)
    return s.dumps({'reset':self.id}).decode('utf-8')
```

**重置密码表单**

```python
#auth/forms.py 已验证邮箱，重置密码表单
class PasswordResetForm(FlaskForm):
    password = StringField('新密码',validators=[DataRequired(),
                   EqualTo('password2',message='两次输入密码必须一致')])
    password2= StringField('确认密码',validators=[DataRequired()])
    submit = SubmitField('重置密码')
```

**确认重设密码路由**

```python
#auth/views.py 点击邮件地址，确认重设密码
@auth.route('reset/<token>',methods=['GET','POST'])
def password_reset(token):
    if not current_user.is_anonymous:
        return redirect(url_for('main.index'))
    form = PasswordResetForm()
    if form.validate_on_submit():
        if User.reset_password(token,form.password.data):
            db.session.commit()
            flash('密码重置成功')
            return redirect(url_for('auth.login'))
        else:
            return redirect(url_for('main.index'))
    return render_template('auth/reset_password.html',form=form)
```

**检验重置密码的token**

```python
v检验重置密码token
@staticmethod
def reset_password(token,new_password):
    s = Serializer(current_app.config['SECRET_KEY'])
    try:
        data = s.loads(token.encode('utf-8'))
    except:
        return False
    user = User.query.get(data.get('reset'))
    if user is None:
        return False
    user.password = new_password
    db.session.add(user)
    return True
```

### 修改电子邮箱地址

需要修改电子邮箱时。

**更改电子邮箱表单**

```python
#auth/forms.py 更改邮箱表单
class ChangeEmailForm(FlaskForm):
    email = StringField('新邮箱',validators=[DataRequired(),length(1,64),Email()])
    password = PasswordField('密码',validators=[DataRequired()])
    submit = SubmitField('更改邮箱')

    def validate_email(self,field):
        if User.query.filter_by(email=field.data).first():
            raise ValidationError('该邮箱已被注册，请重新输入')
```

**更改电子邮箱路由**

```python
#auth/views.py 更改电子邮件路由
@auth.route('change_email',methods=['GET','POST'])
@login_required
def change_email_request():
    form = ChangeEmailForm()
    if form.validate_on_submit():
        if current_user.verify_password(form.password.data):
            new_email = form.email.data
            token = current_user.generate_email_change_token(new_email)
            send_email(new_email,'请确认您的邮箱地址',
                       'auth/email/change_email',user=current_user,token=token)
            flash('我们已向您发送一封电子邮件，其中包含确认新电子邮件地址的说明')
            return redirect(url_for('main.index'))
        else:
            flash('邮箱已被注册或者密码错误！')
    return render_template('auth/change_email.html',form=form)
```

**产生更改电子邮箱的token**

```python
#app/models.py 产生更改邮箱的token
def generate_email_change_token(self,new_email,expiration=3600):
    s = Serializer(current_app.config['SECRET_KEY'],expiration)
    return s.dumps({'change_email':self.id,'new_email':new_email}).decode('utf-8')
```

**检验更改电子邮箱的token**

```python
#app/models.py 检验更改邮箱token
def change_email(self,token):
    s = Serializer(current_app.config['SECRET_KEY'])
    try:
        data = s.loads(token.encode('utf-8'))
    except:
        return False
    if data.get('change_email') != self.id:
        return False
    new_email = data.get('new_email')
    if new_email is None:
        return False
    if self.query.filter_by(email=new_email).first() is not None:
        return False
    self.email = new_email
    #如果更改电子邮箱，这里的电子邮箱的avatar_hash值也要更改，后面详细说明头像Hash
    self.avatar_hash = self.gravatar_hash()
    db.session.add(self)
    return True
```

**确认更改电子邮箱路由**

```python
#auth/views.py 确认更改电子邮箱
@auth.route('change_email<token>')
@login_required
def change_email(token):
    if current_user.change_email(token):
        db.session.commit()
        flash('恭喜您邮箱修改成功！')
    else:
        flash('请求错误！')
    return redirect(url_for('main.index'))
```

