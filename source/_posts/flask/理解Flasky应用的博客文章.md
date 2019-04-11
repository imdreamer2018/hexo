title: 理解Flasky应用的博客文章
author: 追梦人

toc: true

tags:

  - python
  - Flask
categories: []
date: 2019-04-11 15:48:00

---

# 博客文章

本文将实现Flasky的主要功能，即允许用户阅读和撰写博客文章。本章将说明一些新技术：重用模板、分页显示长列表、以及处理富文本。

<!-- more -->

## 提交和显示博客文章

为支持博客文章，我们需要创建一个新的数据库模型。

```python
#app/models.py Post模型
class Post(db.Model):
    __tablename__ = 'posts'
    id = db.Column(db.Integer,primary_key=True)
    body = db.Column(db.Text)
    timestamp = db.Column(db.DateTime,index=True,default=datetime.utcnow)
    author_id = db.Column(db.Integer,db.ForeignKey('users.id'))
    
class User(UserMixin,db.Model):
    #...
    posts = db.relationship('Post',backref='author',lazy='dynamic')
```

```python
#app/main/forms.py 博客文章表单
class PostForm(FlaskForm):
    body = TextAreaField('有什么想说的？',validators=[DataRequired()])
    submit = SubmitField('提交')
```

```python
#app/main/views.py 处理博客文章的首页路由
@main.route('/',methods=['GET','POST'])
def index():
    form = PostForm()
    if current_user.can(Permission.WRITE) and form.validate_on_submit():
        post = Post(body=form.data,author=current_user._get_current_object())
        db.session.add(post)
        db.session.commit()
        return redirect(url_for('.index'))
    posts = Post.query.order_by(Post.timestamp.desc()).all()
    return render_template('index.html',form=form,posts=posts)
```

注意，新文章对象的author属性值为表达式`current_user._get_current_object()`。变量`current_user`由Flask-Login提供，与所有上下文变量一样，也是实现为线程内的代理对象。这个对象的表现类似用户对象，但实际上却是一个轻度包装，包含真正的用户对象。数据库需要真正的用户对象，因此要在代理上调用`_get_current_object()`方法。

## 在资料页中显示博客文章

我们可以改进一下用户资料页面，在上面显示该用户发布的博客文章列表。

```python
#app/main/views.py 获取博客文章的资料页面路由
@main.route('/user/<username>')
def user(username):
    user = User.query.filter_by(username=username).first()
    if user is None:
        abort(404)
    posts = Post.query.order_by(Post.timestamp.desc()).all()
    return render_template('user.html',user=user,posts=posts)
```

## 分页显示长博客文章列表

随着网站的发展，博客文章的数量会不断增多。如果在首页和资料页显示全部文章，页面加载速度会变慢，而且有点不切实际。在Web浏览器中，内容多的网页需要花费更多的时间生成，下载和渲染，因此网页内容变多会让用户体验变差。这一问题的解决方法是**分页**显示数据并分段渲染。

### 创建虚拟博客文章数据

想实现博客文章分页，就需要一个包含大量数据的测试数据库。手动添加数据库记录费时费力，所以最好能使用自动化方案。由多个Python包可用于生成虚拟信息。其中功能相对完善的是Faker。这个包使用pip安装。

```shell
(venv) $ pip install faker
```

严格来说，Faker包并不是这个应用的依赖，因为它在开发过程中使用。为了区分生产环境的依赖和开发环境的依赖，我们可以使`requirements`子目录替换`requirements.txt`文件。在该目录中分别存储不同环境中的依赖，再创建一个`dev.txt`文件，列出开发过程中所需德依赖，再创建一个`prod.txt`文件，列出生产环境所需的依赖。由于两个环境所需的依赖大部分是相同的，可以创建一个`common.txt`文件，在`dev.txt`和`prod.txt`中使用-r参数将其导入。

```
#requirements/dev.txt 开发需求文件
-r common.txt
faker==1.0.4
```

```python
#app/fake.py 生成虚拟用户和博客文章
from random import randint
from sqlalchemy.exc import IntegrityError
from faker import Faker
from . import db
from .models import User,Post

def users(count=100):
    fake = Faker()
    i = 0
    while i < count:
        u = User(email = fake.email(),
                 username = fake.user_name(),
                 password = 'password',
                 confirmed = True,
                 name = fake.name(),
                 location = fake.city(),
                 about_me = fake.text(),
                 member_since = fake.past_date())
        db.session.add()
        try:
            db.session.commit()
            i+=1
        except IntegrityError:
            db.session.rollback()

def posts(count=100):
    fake = Faker()
    user_count = User.query.count()
    for i in range(count):
        u = User.query.offset(randint(0,user_count - 1)).first()
        p = Post(body=fake.text(),
                 timestamp=fake.past_date(),
                 author=u)
        db.session.add(p)
    db.session.commit()
```

这些虚拟对象的属性使用Faker包提供的随机信息生成器生成，可以生成看起来很逼真的姓名、电子邮箱地址、句子等等。

用户的电子邮箱地址和用户名必须是唯一的，但Faker是随机生成这些信息的，因此有重复的风险。如果发生了这种情况，提交数据库会话时会抛出`IntegrityError`异常。此时，数据库会话会回滚，取消添加重复用户的尝试。

```shell
(venv) $ flask shell
>>> from app import fake
>>> fake.users(100)
>>> fake.posts(100)
```

### 在页面中渲染数据

```python
#app/main/views.py 分页显示博客文章列表
@main.route('/',method=['GET','POST'])
def index():
    #...
    page = request.args.get('page',1,type=int)
    pagination = Post.query.order_by(Post.timestamp.desc()).paginate(
    	page,per_page=current_app.conifg['FLASKY_POSTS_PER_PAGE'],
    	error_out=False)
    posts = pagination.items
    return render_template('index.html',form=from,posts=posts,pagination=pagination)
```

渲染的页数从亲求的查询字符串（request.args）中获取，如果没有明确指定，则默认只渲染第1页。参数`type=int`确保参数在无法转换成整数时返回默认值。

为了显示某页中的记录，查询对象最后不能调用all()方法了，现在要调用`Flask-SQLAlchemy`提供的`paginate()`方法。`paginate()`方法的第一个参数——也是唯一必须的参数——是页数。可选参数`per_page`指定每页显示的记录数量；如果没有指定，则默认显示20个记录。另一个可选参数为`error_out`，如果设为True，则请求页数超过范围时返回404错误；如果设为False，则页数超过范围时返回一个空列表。

这样修改之后，首页中的文章列表会只显示有限数量的文章。若想查看第2页的文章，则要在浏览器地址栏中的URL后加上查询字符串`?page=2`。

### 添加分页导航

`paginate()`方法的返回值是一个`Pagination`类对象，这个类在`Flask-SQLAlchemy`中定义。这个对象包含很多属性，用于在模板中生成分页链接，因此将其作为参数传入了模板。

**Flask-SQLAlchemy分页对象的属性**

| 属性     | 说明                   |
| -------- | ---------------------- |
| items    | 当前页面中的记录       |
| query    | 分页的源查询           |
| page     | 当前页数               |
| prev_num | 上一页的页数           |
| next_num | 下一页的页数           |
| has_next | 如果有下一页，值为True |
| has_prev | 如果有上一页，值为True |
| pages    | 查询得到的总页数       |
| per_page | 每页显示的记录数量     |
| total    | 查询返回的记录数量     |

**Flask-SQLAlchemy分页对象的方法**

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| iter_pages(left_edge=2,left_current=2,right_current=5,right_edge=2) | 一个迭代器，返回一个在分页导航中显示的页数列表。这个列表的最左边显示left_edge页，当前页的左边显示left_current页,当前页的右边显示right_current页，最右边显示right_edge页。 |
| prev()                                                       | 上一页的分页对象                                             |
| next()                                                       | 下一页的分页对象                                             |

## 使用Markdown和Flask-PageDown支持富文本文章

对于发布短消息和状态更新来说，纯文本足够用了，但如果用户想发布长文章，就会觉得在格式上收到了限制。本节要将输入文章的多行文本输入框升级，让其支持Markdown句法，还要添加富文本文章的预览功能。

实现这一功能要用到一些新包。

- PageDown：使用JavaScript实现的客户端Markdown到HTML转换程序。
- Flask-PageDown：为Flask包装的PageDown，把PageDown集成到Flask-WTF表单中。
- Markdown：使用Python实现的服务器端Markdown到HTML转换程序。
- Bleach：使用Python实现的HTML清理程序。

这些Python包可使用pip安装：

```shell
(venv) $ pip install flask-pagedown markdown bleach
```

### 使用Flask-PageDown

`Flask-PageDown`扩展定义了一个`PageDownField`类，这个类和`WTForms`中的`TextAreaField`接口一致，使用`PageDownField`字段之前，先要初始化扩展。

```python
#app/__init__.py 初始化Flask-PageDown
from flask_pagedown import PageDown
#...
pagedown = PageDown()
#...
def create_app(config_name):
    #...
    pagedown.init_app(app)
    #...
```

```python
#app/main/forms.py 支持MarkDown的文章表单
from flask_pagedown.fields import PageDownField

class PostForm(FlaskForm):
    body = PageDownField('有什么想说的？',validators=[Required()])
    submit = SubmitField('提交')
```

### 在服务器端处理富文本

提交表单后，POST请求只会发送纯Markdown文本，页面显示的HTML预览会被丢掉。随表单一起发送生成的HTML预览有安全隐患，因为攻击者能很轻松的修改HTML代码，使其和Markdown源不匹配，然后再提交表单。为了安全起见，应该只提交Markdown源文本，然后在服务器上使用Markdown将其转换成HTML。得到HTML后，再使用Bleach进行清理，确保其中只包含几个允许使用的HTML标签。

```python
#app/models.py 在Post模型中处理Markdown文本
from markdown import markdown
import bleach

class Post(db.Model):
    #...
    body_html = db.Column(db.Text)
    #...
    '''
    定义静态方法，将传入的markdown文本转换为HTML
    allowed_tags为HTML标签白名单
    转换过程：
    1、markdown函数初步把Markdown文本转换成HTML
    2、把得到的结果和允许使用的HTML标签列表传给clean函数，clean()函数删除所有不在白名单中的标签
    3、最后一步由linkify()完成，把纯文本中的URL转换成合适的<a>链接。
    '''
    @staticmethod
    def on_change_body(target,value,oldvalue,initiator):
         attrs = {
            '*': ['class'],
            'a': ['href', 'rel'],
            'img': ['alt','src'],
        }
        allowed_tags = ['a','abbr','acronym','b','blockquote','code',
                        'img','src','alt','br','em','i','li','ol','pre',
                        'strong','ul','h1','h2','h3','p']
        target.body_html = bleach.linkify(bleach.clean(markdown(
            			value,output_format='html'),tags=allowed_tags,
                       strip=True,attributes=attrs,protocols=['http', 'https']))
      
#on_change_body()函数注册在body字段上，是SQLAlchemy "set"事件的监听程序，当body字段设了新值，这个函数就会自动调用
db.event.listen(Post.body,'set',Post.on_change_body)    
```

## 博客文章的固定链接

用户有时希望能在社交网络中和朋友分享某篇博客文章的链接。为此，每篇文章都要有一个专页，使用唯一的URL引用。

```python
#app/main/views.py 为文章提供固定链接
def post(id):
    post = Post.query.get_or_404(id)
    return render_template('post.html',posts=[post])
```

## 博客文章编辑器

```python
#app/main/views.py 编辑博客文章路由
@main.route('/edit/<int:id>',methods=['GET','POST'])
@login_required
def edit(id):
    post = Post.query.get_or_404(id)
    if current_user != post.author and not current_user.can(Permission.ADMIN):
        abort(403)
    form = PostForm()
    if form.validate_on_submit():
        post.body = form.body.data
        db.session.add(post)
        db.session.commit()
        flash('博客文章修改成功！')
        return redirect(url_for('.post',id=post.id))
    form.body.data = post.body
    return render_template('edit_post.html',form=form)
```