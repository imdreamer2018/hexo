title: 理解Flasky应用的用户资料
author: 追梦人

toc: true

tags:

  - python
  - Flask
  - UserProfile
categories: []
date: 2019-04-09 15:11:00

---

# 用户资料

本文将为Flasky实现用户资料页面。所有社交网站都会给用户提供资料页面，简要显示用户在网站中的活动情况。用户可以把这个资料页面的URL分享给别人，告诉别人自己在这个网站上。

<!-- more -->

## 资料信息

为了让用户的资料页面更吸引人，可以在数据库中存储用户的一些额外信息。

```python
#app/models.py 用户信息字段
class User(UserMixin,db.Model):
    __tablename__ = 'users'
    #...
    #真实姓名，所在地区，关于我，注册时间，上次登陆时间
    name = db.Column(db.String(64))
    location = db.Column(db.String(64))
    about_me = db.Column(db.Text())
    member_since = db.Column(db.DateTime(),default = datetime.utcnow)
    last_seen = db.Column(db.DateTime(),default = datetime.utcnow)
```

```python
#app/models.py 刷新用户的最后登陆时间
class User(UserMixin,db.Model):
    #...
    def ping(self):
        self.last_seen = datetime.utcnow()
        db.session.add(self)
        db.session.commit()
```

```python
#app/auth/views.py 更新已登陆用户的最后访问时间
@auth.before_app_request
def before_request():
    if current_user.is_authenticated:
        current_user.ping()
    if current_user.is_authenticated and not current_user.confirmed \
        and request.blueprint !='auth' and request.endpoint !='static':
        return redirect(url_for('auth.unconfirmed'))
```

## 用户资料页面

```python
#app/main/views.py 资料页面的路由
@main.route('/user/<username>')
def user(username):
    user = User.query.filter_by(username = username).first_or_404()
    return render_template('user.html',user=user)
```

## 资料编辑器

用户资料的编辑器分两种情况。最显而易见的情况是，用户进入一个页面，输入自己的资料，一遍显示在自己的资料页面上。还有一种不太明显也同样重要的情况，那就是要让管理员能够编辑任意用户的资料。

### 用户级资料编辑器

```python
#app/main/forms.py 定义普通用户资料编辑表单
class EditProfileForm(FlaskForm):
    name = StringField('真实姓名',validators=[Length(0,64)])
    location = StringField('所在地址',validators=[Length(0,64)])
    about_me = TextAreaField('关于我')
    submit = SubmitField('提交')
```

```python
#app/main/views.py 资料编辑路由
@main.route('/edit-profile',methods=['GET','POST'])
@login_required
def edit_profile():
    form = EditProfileForm()
    if form.validate_on_submit():
        current_user.name = form.name.data
        current_user.location = form.location.data
        current_user.about_me = form.about_me.data
        #current_app._get_current_object()赋值给app作为当前程序的实例
        db.session.add(current_user._get_current_object())
        db.session.commit()
        flash('您的资料已更新！')
        return redirect(url_for('.user',username=current_user.username))
    form.name.data = current_user.name
    form.location.data = current_user.location
    form.about_me.data = current_user.about_me
    return render_template('edit_profile.html',form=form)
```

### 管理员级资料编辑器

```python
#app/main/forms.py 定义管理员用户编辑资料表单
class EditProfileAdminForm(FlaskForm):
    email = StringField('电子邮箱',validators=[DataRequired(),Length(1,64),Email()])
    username = StringField('用户名',validators=[
        DataRequired(),Length(1,64),
        Regexp('^[A-Za-z][A-Za-z0-9_]*$',0,'用户名必须仅由字母，数字和下划线组成')])
    confirmed = BooleanField('确认状态')
    role = SelectField('角色',coerce=int)
    name = StringField('真实姓名', validators=[Length(0, 64)])
    location = StringField('所在地址', validators=[Length(0, 64)])
    about_me = TextAreaField('关于我')
    submit = SubmitField('提交')

    def __init__(self,user,*args,**kwargs):
        super(EditProfileAdminForm,self).__init__(*args,**kwargs)
        self.role.choices = [(role.id,role.name)
                             for role in Role.query.order_by(Role.name).all()]
        self.user = user

    def validate_email(self,field):
        if field.data != self.user.email and \
        		User.query.filter_by(email = field.data).first():
            raise ValidationError('该电子邮箱已被注册，请重新输入！')

    def validate_username(self,field):
        if field.data != self.user.username and \
       		 	User.query.filter_by(username = field.data).first():
            raise ValidationError('该用户名已被注册，请重新输入！')
```

`SelectField`是`WTForms`对HTML表单控件`<select>`的包装，功能是实现下拉列表，这个表单中用于选择用户角色。`SelectField`实例必须在其`choices`属性中设置各选项。选项必须是一个由元组构成的列表，各元组都包含两个元素：选项的标识符，以及显示在控件的中的文本字符串。choices列表在构造函数中设定，其值从Role模型中获取，使用一个查询按照角色名的字母顺序排列所有角色。元组中的标识符是角色的id，因为这是个整数，所以在`SelectField`构造函数中加上`coerce=int`参数，把字段的值转换为整数，而不使用默认的字符串。

```Python
#app/main/views.py 管理员的资料编辑路由
@main.route('/edit-profile/<int:id>',methods=['GET','POST'])
@login_required
@admin_required
def edit_profile_admin(id):
    user = User.query.get_or_404(id)
    form = EditProfileAdminForm(user=user)
    if form.validate_on_submit():
        user.email = form.email.data
        user.username = form.username.data
        user.confirmed = form.confirmed.data
        user.role = Role.query.get(form.role.data)
        user.name = form.name.data
        user.location = form.location.data
        user.about_me = form.about_me.data
        db.session.add(user)
        db.session.commit()
        flash('用户资料已更改!')
        return redirect(url_for('.user',username = user.username))
    form.email.data = user.email
    form.username.data = user.username
    form.confirmed.data = user.confirmed
    form.role.data = user.role.id
    form.name.data = user.name
    form.location.data = user.location
    form.about_me.data = user.about_me
    return render_template('edit_profile.html',form=form,user=user)
```

## 用户头像

为了进一步改进资料页面的外观，可以在页面中显示用户的头像。Gravator是一个行业领先的头像服务，能把头像和电子邮件地址关联起来。用户要先到https://en.gravatar.com/中注册账户，然后上传图像。这个服务通过一个特殊的URL对外开放用户的头像，这个URL中包含用户电子邮箱地址的MD5散列值。

**Gravatar查询字符串参数**

| 参数名 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| s      | 图像尺寸，单位为像素                                         |
| r      | 图像级别，可选值由"g","pg","r"和"x"                          |
| d      | 尚未注册Gravatar服务的用户使用默认图像生成方式，可选值有：“404”，返回404错误；一个URL，指向默认图像，某种图像生成方式，包括"m","identicon","monsterid","wavatar","retro","blank" |
| fd     | 强制使用默认头像                                             |

```python

```

生成头像时要生成MD5散列值，这是一项CPU密集型操作，如果要在某个页面中生成大量头像，计算量会非常大，只要电子邮件地址不变，对应的MD5散列值就不会变。鉴于此，我们可以将其缓存在User模型中。

```python
#app/models.py 
class User(UserMixin,db.Model):
    #...
    avatar_hash = db.Column(db.String(32))
    
    def __init__(self.**kwargs):
        #...
        if self.email is not None and self.avatar_hash is None:
            self.avatar_hash = self.gravatar_hash()
            
    def change_email(self,token):
        #...
        self.email = new_email
        self.avatar_hash = self.gravatar_hash()
        db.session.add(self)
        return True
    
    #产生Gravatar哈希值
    def gravatar_hash(self):
        return hashlib.md5(self.email.lower().encode('utf-8')).hexdigest()


    #生成Gravatar头像
    def gravatar(self,size=100,default='wavatar',rating='g'):
        g = {'1':'mm','2':'identicon','3':'monsterid','4':'wavatar','5':'retro'}
        if request.is_secure:
            url = 'https://secure.gravatar.com/avatar'
        else:
            url = 'http://www.gravatar.com/avatar'
        hash = self.avatar_hash or self.gravatar_hash()
        default = g[str(randint(1,5))]
        return '{url}/{hash}?s={size}&d={default}&r={rating}'.format(
            url=url,hash=hash,size=size,default=default,rating=rating)
```

