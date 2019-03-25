title: 初见Flask应用的基础结构
author: 追梦人

toc: true

tags:

  - python
  - flask
categories: []
date: 2019-03-25 22:39:00

---

# 应用基础结构

## 初始化

所有Flask应用都必须创建一个应用实例。Web服务器使用一种名为Web服务器网关接口(WSGI)的协议，把接受来自客户端的所有请求都交给这个对象处理。应用实例是Flask类的对象，通常由下述代码创建。

```python
from flask import Flask
app = Flask(__name__)
```

## 路由和视图函数

客户端把请求发送给Web服务器，Web服务器再把请求发给了Flask应用实例，应用实例需要知道对每个URL的请求要运行哪些代码，所以保存了一个URL到Python函数的映射关系。处理URL和函数之间关系的程序成为**路由**。

Flask中使用实例提供的app.route装饰器。**装饰器**是Python语言的标准特性，惯常用法是采用闭包的方法把函数注册为事件处理程序。

```python
@app.route('/')
def index():
    return '<h1>Hello World!</h1>'
```

通常把index()函数注册成为应用根地址的处理程序，其中index()处理入站请求的函数成为**视图函数**，如果应用部署再域名为www.example.com的服务器上，在浏览器中输入http://www.example.com就会出发服务器执行index()函数。

## 一个完整的应用

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return '<h1>Hello World!</h1>'
```

## Web开发服务器

Flask应用自带Web开发服务器，通过在终端flask run命令启动，这个命令在FLASK_APP环境变量指定的Python脚本中寻找实例，默认情况下是寻找app.py。如果你的应用实例是hello.py，则需要指定应用实例。

```
#Linux环境下设置应用实例
export FLASK_APP = hello.py
#Windows环境下设置应用实例
set FLASK_APP = hello.py
#启动Flask的Web服务器
flask run
```

服务器运行后，在浏览器中输入http://localhost:5000/就可以看到页面运行结果了。

如果不想通过flask run命令来启动Web服务，可以在应用实例中加入如下代码

```python
if __name__ == '__main__':
    app.run()
```

## 动态路由

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return '<h1>Hello World!</h1>'

@app.route('/user/<name>')
def user(name):
    return '<h1>hello,{}!</h1>'.format(name)
```

启动Web服务器之后，然后访问http://localhost:5000/user/Lola。这时就会返回一个hello,Lola!的结果，其中<name>就是传递动态参数，并且在函数user()中加入参数user(name)，format是Python的一个参数匹配，format(value)，value匹配前面字符串中的{}，类似C语言中的%d,%s等等。

## 调试模式

Flask应用可以在调试模式中运行，在这个模式下，开发服务器会默认加载两个工具：**重载器**和**调试器**

开启调试模式只需在终端中输入。

```
#Linux环境下设置应用实例
export FLASK_DEBUG = 1
#Windows环境下设置应用实例
set FLASK_DEBUG = 1
```

```python
if __name__ == '__main__':
    app.run(debug = True)
```

## 请求—响应循环

### 应用和请求上下文

Flask从客户端收到请求时，要让视图函数能访问一些对象，这样才能处理请求。要想让视图函数能够访问请求对象，一种直截了当的方法就是将参数传入视图函数，不过这样会导致应用中的每个视图函数都多出一个参数，如果视图函数还要访问其他对象，就又需要一个参数，这样就会乱糟糟。

为了避免大量可有可无的参数把视图函数搞得乱七八糟，Flask使用上下文临时把某些对象变为全局可访问。

```python
from flask import request
app = Flask(__name__)

@app.route('/')
def index():
    user_agent = request.headers.get('User-Agent')
    return '<p>Your browser is {}</p>'.format(user_agent)
```

在这个视图函数中，把request当作全局变量使用，事实上request不可能是全局变量，因为在多线程服务器中，多个线程同时处理不同客户端发送的不同请求时，每个线程看到的request对象必然不同，Flask使用上下文让特定的变量在一个线程中全局可访问，却同时不会干扰到其他线程。

**Flask上下文全局变量**

| 变量名      | 上下文     | 说明                                                   |
| ----------- | ---------- | ------------------------------------------------------ |
| current_app | 应用上下文 | 当前应用的应用实例                                     |
| g           | 应用上下文 | 处理请求时用作临时存储的对象，每次请求都会重设这个变量 |
| request     | 请求上下文 | 请求对象，封装了客户端发出的HTTP请求中的内容           |
| session     | 请求上下文 | 用户会话，值为一个字典，存储请求之间需要”记住“的值     |

### 请求分派

应用收到客户端发来的请求时，要找到处理请求的视图函数，为了完成这个任务，Flask会在应用的**URL映射**中查找请求的URL。

### 请求对象

**Flask请求对象**(部分常用属性或方法，完整请查阅Flask文档)

| 属性或方法  | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| form        | 一个字典，存储请求提交的所有表单字段                         |
| args        | 一个字典，存储通过URL查询字符串传递的所有参数                |
| cookies     | 一个字典，存储请求的所有cookie                               |
| headers     | 一个字典，存储请求的所有HTTP首部                             |
| files       | 一个字典，存储请求上传的所有文件                             |
| get_json    | 返回一个Python字典，包含解析请求主体后得到的JSON             |
| endpoint    | 处理请求的Flask端点的名称，Flask把视图函数的名称作路由端点的名称 |
| method      | HTTP请求方法                                                 |
| is_secure() | URL方案（http或https）                                       |

### 请求钩子

有时在处理请求之前或者之后执行代码会很有用。比如在请求开始时，我们需要创建数据库连接或者验证发起请求的用户身份。Flask提供了注册通用函数的功能。

| 装饰器方法            | 说明                                                     |
| --------------------- | -------------------------------------------------------- |
| @before_request       | 注册一个函数，在每次请求之前运行                         |
| @before_first_request | 注册一个函数，只在处理第一个请求之前运行                 |
| @after_request        | 注册一个函数，如果没有未处理的异常抛出，在每次请求后运行 |
| @teardown_request     | 注册一个函数，即使有未处理的异常抛出，也在每次请求后运行 |

### 响应

Flask调用视图函数后，会将其返回值作为响应的内容，多数情况下，响应就是一个字符串，作为HTML页面会送客户端。如果视图函数返回的响应需要不同的状态码，可以把数字代码作为第二个返回值加到响应文本之后，视图函数返回的响应还可以接受第三个参数，这是一个由HTTP响应首部组成的字典。

如果不想返回有1个，2个，3个值组成的元组，Flask视图函数还可以返回一个响应对象，make_response()函数可以接受1个，2个，3个参数，然后返回一个等效的响应对象。

```python
from flask import make_response

@app.route('/')
def index():
    response = make_response('<h1>This document carries a cookie!</h1>')
    response.set_cookie('answer','42')
    return response
```

**Flask响应对象**

| 属性和方法      | 说明                                         |
| --------------- | -------------------------------------------- |
| status_code     | HTTP数字状态码                               |
| headers         | 一个类似字典的对象，包含随响应发送的所有首部 |
| set_cookie()    | 为响应添加一个cookie                         |
| delete_cookie() | 删除一个cookie                               |
| content_length  | 响应主体的长度                               |
| content_type    | 响应主体的媒体类型                           |
| set_data()      | 使用字符串或者字节值设定响应                 |
| get_data()      | 获取响应主体                                 |

响应有个特殊的类型，成为**重定向**，这种响应没有页面文档，通常告诉浏览器一个新URL，重定向的状态码通常时302，在Location首部提供目标URL。Flask提供了redirect()辅助函数。

```python
from flask import redirect

@app.route('/')
def index():
    return redirect('http://www.example.com')
```

还有一种特殊的响应由abort()函数生成，用于处理错误。下面例子中，如果动态参数id对于的用户不存在，就返回状态码404。并且abort()函数不会把控制权交给其他函数，而是抛出异常。

```python
from flask import abort

@app.route('/user/<id>')
def get_user(id):
    user = load_user(id)
    if not user:
        abort(404)
    return '<h1>hello,{}</h1>'.format(user.name)
```

