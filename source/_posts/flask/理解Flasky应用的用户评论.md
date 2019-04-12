title: 理解Flasky应用的用户评论
author: 追梦人

toc: true

tags:

  - python
  - Flask
categories: []
date: 2019-04-12 21:36:00

---

# 用户评论

允许用户交互是社交博客平台成功的关键。在本文，你将学到如何实现用户评论功能。

<!-- more -->

## 评论在数据库中的表示

评论和博客文章没有太大区别，都有正文、作者和时间戳，而且在这个特定实现中都是用Markdown句法编写。

![Aqt5IU.png](https://s2.ax1x.com/2019/04/12/Aqt5IU.png)

评论属于某篇博客文章，因此定义了一个从`posts`表到`comments`表的一对多关系。使用这个关系可以获取某篇博客文章的评论列表。

`comments`表还与`users`表之间有一对多关系。通过这个关系可以获取用户发表的所有评论，还能间接知道用户发表了多少篇评论。

```python
#app/models.py Comment模型
class Comment(db.Model):
    __tablename__ = 'comments'
    id = db.Column(db.Integer,primary_key=True)
    body = db.Column(db.Text)
    body_html = db.Column(db.Text)
    timestamp = db.Column(db.DateTime,index=True,default=datetime.utcnow)
    disabled = db.Column(db.Boolean)
    author_id = db.Column(db.Integer,db.ForeignKey('users.id'))
    post_id = db.Column(db.Integer,db.ForeignKey('posts.id'))
    
    @staticmethod
    def on_changed_body(target,value,oldvalue,initiator):
        allowed_tags = ['a','abbr','acronym','b','code','em','i','strong']
        target.body_html = bleach.linkify(bleach.clean(
            markdown(value,output_format='html'),tags=allowed_tags,strip=True))

db.event.listen(Comment.body,'set',Comment.on_changed_body)
```

`Comment`模型的属性几乎和`Post`模型一样，不过多了一个`disabled`字段。这是个布尔值字段，协管员通过这个字段查禁不当评论。

为了完成对数据库的修改，`User`和`Post`模型还要建立与`comments`表一对多关系。

```python
#app/models.py users和posts表与comments表之间的一对多关系
class User(UserMixin,db.Model):
    #...
    comments = db.relationship('Comment',backref='author',lazy='dynamic')
  
class Post(db.Model):
    #...
    comments = db.relationship('Comment',backref='post',lazy='dynamic')
```

## 提交和显示评论

```python
#app/main/forms.py 评论输入表单
class CommentForm(FlaskForm):
    body = StringField('输入你的评论',validators=[DataRequired()])
    submit = SubmitField('评论')
```

```python
#app/main/forms.py 支持博客文章评论
@main.route('/post/<int:id>',methods=['GET','POST'])
@login_required
def post(id):
    post = Post.query.get_or_404(id)
    form = CommentForm()
    if form.validate_on_submit():
        comment = Comment(body=form.body.data,
                          post=post,
                          author=current_user._get_current_object())
        db.session.add(comment)
        db.session.commit()
        flash('您已提交评论')
        return redirect(url_for('.post',id=post.id,page=-1))
    page = request.args.get('page',1,type=int)
    if page == -1:
        page = (post.comments.count() - 1) // \
        		current_app.config['FLASKY_COMMENTS_PER_PAGE'] + 1
    pagination = post.comments.order_by(Comment.timestamp.asc()).paginate(
    	page,per_page=current.app.config['FLASKY_COMMENTS_PER_PAGE'],
    	error_out=False)
    comments = pagination.items
    return render_template('post.html',posts=[post],form=form,
                           comments=comments,pagination=pagination)
```

## 管理评论

```python
#app/main/views.py 展示所有评论的路由
@main.route('/moderate')
@login_required
@permision_required(Permission.MODERATE)
def moderate():
    page = request.args.get('page',1,type=int)
    pagination = Comment.query.order_by(Comment.timestamp.desc()).paginate(
    	page,per_page=current_app.config['FLASKY_COMMENTS_PER_PAGE'],
    	error_out=False)
    comments = pagination.items
    return render_template('moderate.html',comments=comments,
                            pagination=pagination,page=page)
```

```python
#app/main/views.py 管理评论路由
@main.route('/moderate/enable/<int:id>')
@login_required
@permission_required(Permission.MODERATE)
def moderate_enable(id):
    comment = Comment.query.get_or_404(id)
    comment.disabled = False
    db.session.add(comment)
    db.session.commit()
    return redirect(url_for('.moderate',page=request.args.get('page',1,type=int)))

@main.route('/moderate/disable/<int:id>')
@login_required
@permission_required(Permission.MODERATE)
def moderate_disable(id):
    comment = Comment.query.get_or_404(id)
    comment.disabled = True
    db.session.add(comment)
    db.session.commit()
    return redirect(url_for('.moderate',page=request.args.get('page',1,type=int)))
```

