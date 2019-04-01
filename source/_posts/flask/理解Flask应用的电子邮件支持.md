title: 理解Flask应用的电子邮件支持
author: 追梦人

toc: true

tags:

  - python
  - Flask
  - Flask-mail
categories: []
date: 2019-04-01 20:02:00

---

# 电子邮件

很多类型的应用都需要在特定时间发生时通知用户，而常用的通信方法是电子邮件。接下来将介绍在Flask应用中发送电子邮件。

<!-- more -->

## 使用Flask-Mail提供电子邮件支持

虽然Python标准库中的smtplib包可用于在Flask应用中发送电子邮件，但包装了smtplib的Flask-Mail扩展能更好地与Flask集成。Flask-Mail使用pip安装：

```shell
(venv) $ pip install flask-mail
```

Flask-Mail连接到简单邮件传输协议（SMTP）服务器，把邮件交给这个服务器发送。如果不进行配置，则Flask-Mail连接Localhost上的25端口，无需验证身份即可发送电子邮件。

**Flask-Mail SMTP服务器的配置**

| 配置          | 默认值    | 说明                           |
| ------------- | --------- | ------------------------------ |
| MAIL_SERVER   | localhost | 电子邮件服务器的主机名或IP地址 |
| MAIL_PORT     | 25        | 电子邮件服务器的端口           |
| MAIL_USE_TLS  | False     | 启动传输层安全（TLS）协议      |
| MAIL_USE_SSL  | False     | 启动安全套接层（SSL）协议      |
| MAIL_USERNAME | None      | 邮件账户的用户名               |
| MAIL_PASSWORD | None      | 邮件账户的密码                 |

在开发过程中，连接到外部SMTP服务器可能更方便。举个例子。由于QQ邮箱总是识别我是垃圾邮件，所以还是使用了阿里的企业邮箱。配置任何邮箱，都可以参考邮箱的配置文件。

```python
#hello.py 配置Flask-Mail使用阿里企业邮箱
import os
#...
#这里的domain.com为自己的域名
app.config['MAIL_SERVER'] = 'smtp.domain.com'
app.config['MAIL_PORT'] = 465
app.config['MAIL_USE_SSL'] = True
app.config['MAIL_USERNAME'] = os.enviorn.get('MAIL_USERNAME')
app.config['MAIL_PASSWORD'] = os.environ.get('MAIL_PASSWORD')
```

这里千万不要把账户凭据直接写入脚本，特别是当你计划开源自己的作品时。为了保护账户信息，脚本应该从环境变量中导入敏感信息。

```shell
(venv) $ set MAIL_USERNAME = ******
(venv) $ set MAIL_PASSWORD = ******
```

**Flask-Mail**初始化方法

```python
from flask_mail import Mail
mail = Mail(app)
```

## 在Python shell中发送电子邮件

```shell
(venv) $ flask shell
>>> from flask_mail import Message
>>> from hello import Mail
>>> msg = Message('test mail',sender='your@example.com',recipients=['your@example.com'])
>>> msg.body = 'This is the plain text body'
>>> msg.html = 'This is the <b>HTML</b> body'
>>> with app.app_context():
...		mail.send(msg)
```

注意，Flask-Mail的send()函数使用current_app，因此要在激活的应用上下文中执行。

## 在应用中集成电子邮件发送功能

```python
# hello.py 电子邮件支持
from flask_mail import Message

app.config['FLASKY_MAIL_SUBJECT_PREFIX'] = '[Flasky]'
app.config['FLASKY_MAIL_SENDER] = 'Flask Admin <flask@example.com>'

def send_email(to,subject,template,**kwargs):
    msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + subject,
                  sender=app.config['FLASKY_MAIL_SENDER'],recipients=[to])
    msg.body = render_template(template + '.txt' ,**kwargs)
    msg.html = render_template(template + '.html' ,**kwargs)
    mail.send(msg)
```

这个函数用到了两个应用层面的配置项，分别定义邮件主体的前缀和发件人的地址。send_email()函数的参数分别为收件人地址、主题、渲染邮件正文的模板和关键字参数列表。

```python
# hello.py 电子邮件示例
#...
app.config['FLASKY_ADMIN'] = os.environ.get('FLASKY_ADMIN')
#...
@app.route('/',methods=['GET','POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        user = User.uqery.filter_by(username = form.name.data).first()
        if user is None:
            user = User(form.name.data)
            db.session.add(user)
            session['know'] = False
            if app.config['FLASKY_ADMIN']:
                send_email(app.config['FLASKY_ADMIN'],'New User',
                           'mail/new_user',user=user)
         else:
            session['know'] = True
         session['name'] = form.name.data
         form.name.data = ''
         return redirect(url_for('index'))
   	return render_template('index.html',form=form,name=session.get('name'),
                          knowm = session.get('known',False)
```

## 异步发送电子邮件

如果你发送了几封电子邮件，可能会注意到mail.send()函数在发送电子邮件时停滞了几秒钟，在这个过程中浏览器就像无响应一样，为了在处理过程中避免不必要的延迟，我们可以把发送电子邮件的函数移到后台线程中。

```python
from threading import Thread

def send_async_email(app,msg):
    with app.app_context():
        mail.send(msg)
        
def send_email(to,subject,template,**kwargs):
    msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + subject,
                  sender=app.config['FLASKY_MAIL_SENDER'],recipients=[to])
    msg.body = render_template(template + '.txt' ,**kwargs)
    msg.html = render_template(template + '.html' ,**kwargs)
    thr = Thread(target=send_async_email,args=[app,msg])
    thr.start()
    return str
```

