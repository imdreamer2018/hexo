title: 理解Flask应用的Web表单
author: 追梦人

toc: true

tags:

  - python
  - Flask
  - WebForm
categories: []
date: 2019-03-26 21:06:00

---

# Web表单

使用HTML可以创建Web表单，供用户填写信息，表单数据由Web浏览器提交给服务器，这一过程通常使用POST请求，对于包含表单数据的POST请求来说，用户填写的信息通过request.form访问。

Flask-WTF扩展可以把处理Web表单的过程编程一种愉悦的体验。Flask-WTF及其依赖可以使用pip安装：

```
(venv)$ pip install flask-wtf
```

<!-- more -->

## 配置

与其他多数扩展不同，Flask-WTF无须再应用层初始化，但是它要求配置一个**密钥**。密钥是一个由随机字符构成的唯一字符串，通过加密或签名以不同的方式提升应用的安全性。Flask使用这个密钥保护用户会话，以防被篡改。

```
#hello.py 配置Flask-WTF
app = Flask(__name__)
app.config['SECRET_KEY']= 'hard to guess string'
```

app.config字典可用于存储Flask、扩展和应用自身的配置变量。

Flask-WTF之所以要求应用配置一个密钥，是为了防止表单遭到跨站请求伪造(CSRF)攻击。恶意网站把请求发送到被攻击者已登陆的其他网站时，就会引发CSRF攻击。Flask-WTF为所有表单生成安全令牌，存储在用户会话中，令牌是一种加密签名，根据密钥生成。

## 表单类

使用Flask-WTF时，在服务器端，每个Web表单都由一个继承来自FlaskFrom的类表示，这个类定义表单中的一组字段，每个字段都用对象表示，字段对象可附属一个或多个**验证函数**。

```python
# hello.py 定义表单类
from flask_wrf import FlaskFrom
from wtforms import StringField,SubmitField
from wtforms.validators import DataRequired

class NameForm(FlaskForm):
    name = StringField('what is your name?',validators=[DataRequired()])
    submit = SubmitField('Submit')
```

**WTForms支持的HTML标准字段**

![AUI1bt.png](https://s2.ax1x.com/2019/03/26/AUI1bt.png)

**WTForms验证函数**

![AUIN8g.png](https://s2.ax1x.com/2019/03/26/AUIN8g.png)

## 把表单渲染成HTML

Flask-Bootstrap扩展提供了一个高层级的辅助函数，可以使用Bootstrap预定义的表单样式渲染整个Flask-WTF表单。

```jinja2
{% import "bootstrap/wtf.html" as wtf %}
{{ wtf.quick_form(form) }}
```

导入的bootstap/wtf.html文件中定义了一个使用Bootstrap渲染Flask-WTF表单对象的辅助函数。wtf.quick_form()函数的参数为Flask-WTF表单对象。

## 在视图函数中处理表单

如下视图函数index()有两个任务：一个是渲染表单，二是接收用户在表单中填写的数据。

```python
@app.route('/',methods=['GET','POST'])
def index():
    name = None
    form = NameForm()
    if form.validate_on_submit():
        name = form.name.data
        form.name.data = ''
    return render_template('index.html',from=from,name=name)
```

app.route装饰器中多出的methods参数告诉Flask，在URL映射中把这个视图函数注册为GET和POST请求的处理程序。如果没有指定这个参数，默认为GET请求。

如果数据能被所有验证函数所接受，那么validate_on_submit()方法返回值为True，否则为False。用户首次访问时，服务器会收到一个没有表单数据的GET请求，所以validate_on_submit()为False，此时跳过if判断，将空的from表单和空的name值作为参数，用户会看到浏览器中显示了一个表单。

## 重定向和用户会话

在上一个hello.py存在一个可用性问题，用户输入名字后提交表单，然后点击浏览器的刷新，会看到一个警告，要求再次提交表单之前进行确认，因为刷新页面时浏览器会重新发送之前发送过的请求。如果前一个请求时包含表单数据的POST请求，刷新页面后会再次提交表单，这并不是我们想执行的操作。

我可以使用**重定向**作为POST请求的响应，响应内容为URL，而不是HTML代码的字符串。浏览器收到这种响应时，会向重定向的URL发起GET请求，显示页面内容，这个页面加载可能需要多花几毫秒，因为要先把第二个请求发给服务器。

但这种方法又会产生另一个问题，应用处理POST请求之后，可以通过form.name.data获取用户输入的名字，然而一旦这个请求结束，数据也不见了。为此，我们可以把数据存储在**用户会话**中。用户会话是一种私有存储，每个连接到服务器的客户端都可以访问。它是请求上下文的变量，名为session。

```python
#hello.py 重定向和用户会话

@app.route('/',methods=['GET','POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        session['name'] = form.name.data
        return redirect(url_for('index'))
    return render_template('index.html',form = from,name = session.get['name'])
```

## 闪现消息

请求完成后，有时需要让用户知道状态发生了变化。Flask本身内置这个功能，flash()函数可以实现这种效果。

```python
#hello.py 闪现消息

@app.route('/',methods=['GET','POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        old_name = session.get['name']
        if old_name is not None and old_name !=form.name.data:
            flash('Looks like you have changed your name!')
        session['name'] = form.name.data
        return redirect(url_for('index'))
    return render_template('index.html',form = from,name = session.get['name'])
```

但仅调用flask()函数并不能把消息显示出来，应用的模板必须渲染这些消息。最好在基模板中渲染闪现消息，因为这样所有页面都可以显示需要显示的消息。Flask把get_flashed_message()函数开放给模板，用户获取并渲染闪现消息。

