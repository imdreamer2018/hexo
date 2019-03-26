title: 理解Flask应用中的模板
author: 追梦人

toc: true

tags:

  - python
  - Flask
categories: []
date: 2019-03-26 19:24:00

---

# 模板

## Jinja2模板引擎

形式最简单的Jinja2模板就是一个包含响应文本的文件，如下代码所示。

```jinja2
#templates/index.html
<h1>Hello World!</h1>
#templates/user.html
<h1>Hello,{{name}}</h1>
```

<!-- more -->

### 渲染模板

默认情况下，Flask在应用目录中的templates子目录中寻找模板。Flask提供的render_template()函数把Jinja2模板引擎集成到应用中，这个函数第一个参数是模板的文件名，随后的参数都是键值对。

```python
from flask import Flask,render_template
#...

@app.route('/')
def index():
    return render_template('index.html')
@app.route('/user/<name>')
def user(name):
    return render_template('user.html',name = name)
```

### 变量

在上述事例中，模板使用{{name}}结构表示一个变量，这是一种特殊的占位符，告诉模板引擎这个位置的值从渲染模板使用的数据中获取。Jinja2能识别所有类型的变量。例如列表、字典和对象。

变量的值可以用过滤器修改，过滤器添加变量之后，二者之间一竖线分隔，例如。

```jinja2
hello.{{name|capitalize}}
```

**Jinja2变量过滤器**

| 过滤器名   | 说明                                       |
| ---------- | ------------------------------------------ |
| safe       | 渲染值时不转义                             |
| capitalize | 把值得首字母转换成大写，其他字母抓换成小写 |
| lower      | 把值转换成小写形式                         |
| upper      | 把值转换成大写形式                         |
| title      | 把值得每个单词的首字母都转换成大写         |
| trim       | 把值得首尾空格删掉                         |
| striptags  | 渲染之前把值中所有的HTML标签都删掉         |

控制结构

Jinja2提供了多种控制结构，可用来改变模板的渲染流程。

```jinja2
#条件判断语句
{% if user %}
	Hello,{{user}}!
{% else %}
	Hello,Stranger!
{% endif %}

#for循环语句
<ul>
    {% for comment in comments %}
    	<li>{{comment}}</li>
    {% endfor %}
</ul>

#支持宏，类似Python代码中的函数
{% macro render_comment(comment) %}
	<li>{{comment}}</li>
{% endmacro %} 
<ul>
    {% for comment in comments %}
    	<li>{{render_comment(comment)}}</li>
    {% endfor %}
</ul>
#可以把宏单独保存在文件中，使用的时候导入
{% import 'macros.html' as macros %}
```

基模板中定义的区块可以在衍生模板中覆盖。Jinja2使用block和endblock指令在基模板中定义内容区块。

```jinja2
{% extends "base.html"%}
{% block title %}Index{% endblock %}
{% block head %}
	{{super()}}
	<style></style>
{% endblock %}
{% block body %}
<h1>Hello,World!</h1>
{% endblock %}
```

extends指令声明这个模板衍生自base.html，如果基模板和衍射模板中的同名区块中都有内容，衍生模板中的内容将显示出来，在衍生模板的区块里可以调用super()，引用基模块中同名模块里的内容。

## 使用Flask-Bootstrap集成Bootstrap

Flask扩展在创建应用实例时初始化

```python
from flask_bootstrap import Bootstrap
#...
bootstrap = Bootstrap(app)
```

本节涉及到前端的知识，这里不做详细介绍，详情可查阅Bootstrap官方文档。

### 自定义错误页面

如果你在浏览器的地址栏中输入了无效的路由，会看到一个状态码为404的错误页面。Flask允许应用使用模板自定义错误页面，最常见的错误代码是404和500，可以利用app.errorhandler装饰器为这两个错误提供自定义的处理函数。

```python
@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html'),404

@app.errorhandler(500)
def internal_server_error(e):
    return render_template('500.html'),500
```

## 链接

任何具有多个路由的应用都需要可以连接不同页面的链接，例如导航栏。

在模板中直接编写简单路由的URL链接不难，但对于可变动部分的动态路由，在模板中构建就比较困难了。为了避免这些问题，Flask提供了url_for()辅助函数，它使用应用的URL映射中保存的信息生成URL。

url_for()函数最简单的用法是以视图函数名作为参数，返回对应的URL。例如调用url_for('index')得到的结果就是/，即应用的根URL，调用url_for('index',_external=True)返回的则是绝对地址。

```python
url_for('user',name='john',_external=True)
#返回结果是http://localhost:5000/user/john
url_for('user',name='john',page=2,version=1)
#返回结果是/user/john&page=2&version=1
```

## 静态文件

Web应用不是仅有Python代码和模板组成，多数应用还会使用静态文件。默认情况下，Flask在应用根目录中名为static的子目录中寻找静态文件。

```python
url_for('static',filename='css/style.css',_external=True)
#得到的结果是http://localhost:5000/static/css/style.css
```

## 使用Flask-Monment本地化日期和时间

如果Web应用的用户来自世界各地，那么处理日期和时间可不是一个简单的任务。服务器需要统一时间单位，这和用户所在的地理位置无关，一般采用UTC，即本初子午线的时间。

有一个使用JavaScript开发的优秀客户端开源库，名为Moment.js。下面为初始化Flask-Moment的方法。

```python
from flask_moment import Moment
moment = Moment(app)
```

```jinja2
#template/base.html：引入Moment.js库
{% block scripts %}
{{ super() }}
{{ moment.include_moment() }}
{% endblock %}
```

为了处理时间戳，Flask-Moment向模板开放了moment对象，下面事例如何把current_time传入模板进行渲染

```python
from datetime import datetime

@app.route('/')
def index():
    return render_template('index.html',current_time = datetime.utcnow())
```

```jinja2
#templates/index.html:使用Flask-Moment时间戳
<p>The local date and time is {{ moment(current_time).format('LLL') }}</p>
<p>That was {{ moment(current_time).fromNow(refresh=True) }}</p>
```

format('LLL')函数根据客户端计算机中的时区和区域设置渲染日期和时间。参数决定了渲染的方式，从L到LLL分别对应不同的复杂度。

第二行中from_Now()渲染相对时间戳，而且会随着时间的推移自动刷新显示时间，这个时间戳最开始会显示为"a few seconds ago"，但是设定refresh=True参数后，其内容跟就会随着时间推移而刷新。

Flask-Moment渲染时间戳可实现多种语言的本地化，语言可以在模板中选择，默认是英语，例如配置中文的方式如下：

```jinja2
{% block scripts %}
{{ super() }}
{{ moment.include_moment() }}
{{ moment.locale('zh-CN') }}
{% endblock %}
```

