title: 梳理HTTPAuth验证用户身份逻辑
author: 追梦人

toc: true

tags:

  - python
  - Flask
  - RESTful
  - HTTPAuth
categories: []
date: 2019-04-15 15:38:00

---

# 梳理Flasky应用的HTTPAuth验证用户身份逻辑

在上一节博客中，介绍到了Flasky应用编程接口，前面说过，`REST`式`Web`服务的特征之一是无状态，即服务器在两次请求之间不能“记住”客户端的任何信息。客户端必须在发出的请求中包含所有必要信息，因此所有请求都必须包含用户凭据。

`Flasky`应用当前的登陆功能是在`Flask-Login`的帮助下实现的，数据存储在用户会话中。默认情况下，Flask把会话保存在客户端`cookie`中，因此服务器没有保存任何用户相关信息，都转交给客户端保存。这种实现方式看起来遵守了`REST`架构的无状态要求，但在`REST`式`We`b服务中使用`cookie`有点不现实，因为Web浏览器之外的客户端很难提供对`cookie`的支持。鉴于此，在API中使用`cookie`并不是一个很好的设计选择。

<!-- more -->

> `REST`架构的无状态要求看起来似乎获取严格，但这并不是随意提出的要求——无状态的服务器**伸缩**起来更加简单。如果服务器保存了客户端的相关信息，那么必须保证特定客户端发送的请求由同一台服务器处理，或者使用共享存储器客户端数据。这两点都难以实现，但是如果服务器式无状态的，这两个问题就不复存在。

因为`REST`架构基于`HTTP`协议，所以发送凭据的最佳方式式使用**`HTTP`身份验证**，基本验证和摘要验证都可以。在`HTTP`身份验证中，用户凭据包含在每个请求的`Authorization`首部中。

`HTTP`身份验证协议很简单，可以直接实现，不过`Flask-HTTPAuth`扩展提供了一个遍历的包装，把协议的细节隐藏在装饰器之中，类似于`Flask-Login`提供的`login_required`装饰器。



## HTTPAuth验证流程

![Aj0tLF.png](https://s2.ax1x.com/2019/04/15/Aj0tLF.png)

由于每次访问资源都需要登陆验证，所以这里每次访问是需要将用户凭据包含在每个请求的`Authorization`首部中。

## 验证身份模块

这里我们利用了`Flask-HTTPAuth`扩展提供装饰器。当我们需要访问资源的时候，将用户凭据包含在每个请求的`Authorization`首部中，如果没有通过验证就无法访问资源。这里需要在`before_request`处理程序中使用`login_required`装饰器。

### 设置蓝本全局路由保护

```python
#在before_request处理程序中验证身份
@api.before_request
@auth.login_required
def before_request():
    if not g.current_user.is_anonymous and not g.current_user.confirmed:
        return forbidden('Unconifrmed account')
```

其中`login_required`装饰器：

```python
def login_required(self, f):
    @wraps(f)
    def decorated(*args, **kwargs):
        auth = self.get_auth()

        # Flask normally handles OPTIONS requests on its own, but in the
        # case it is configured to forward those to the application, we
        # need to ignore authentication headers and let the request through
        # to avoid unwanted interactions with CORS.
        if request.method != 'OPTIONS':  # pragma: no cover
            password = self.get_auth_password(auth)

            if not self.authenticate(auth, password):
                # Clear TCP receive buffer of any pending data
                request.data
                return self.auth_error_callback()

        return f(*args, **kwargs)
    return decorated
```

首先`self.get_auth()`得到首部`Authorization`。源码如下，这里得到首部字段，并且包装得到`auth`，并返回给`login_required`函数。

```python
def get_auth(self):
    auth = request.authorization
    if auth is None and 'Authorization' in request.headers:
        # Flask/Werkzeug do not recognize any authentication types
        # other than Basic or Digest, so here we parse the header by
        # hand
        try:
            auth_type, token = request.headers['Authorization'].split(
                None, 1)
            auth = Authorization(auth_type, {'token': token})
        except ValueError:
            # The Authorization header is either empty or has no token
            pass

    # if the auth type does not match, we act as if there is no auth
    # this is better than failing directly, as it allows the callback
    # to handle special cases, like supporting multiple auth types
    if auth is not None and auth.type.lower() != self.scheme.lower():
        auth = None

    return auth
```

接下来需要`self.get_auth_password(auth)`得到auth中包含用户的用户密码。然后再调用`self.authenticate(auth, password)`去进行身份验证。

```python
def authenticate(self, auth, stored_password):
    if auth:
        username = auth.username
        client_password = auth.password
    else:
        username = ""
        client_password = ""
    if self.verify_password_callback:
        return self.verify_password_callback(username, client_password)
    if not auth:
        return False
    if self.hash_password_callback:
        try:
            client_password = self.hash_password_callback(client_password)
        except TypeError:
            client_password = self.hash_password_callback(username,
                                                          client_password)
    return client_password is not None and \
        client_password == stored_password
```

这里我们会得到`auth.username`和`auth.password`。如果请求首部不包含`Authorization`，就将`username`和`client_password`设为空值。最后一步就是调用`self.verify_password_callback`回调函数进行验证。

```python
class HTTPBasicAuth(HTTPAuth):
    def __init__(self, scheme=None, realm=None):
        super(HTTPBasicAuth, self).__init__(scheme or 'Basic', realm)

        self.hash_password_callback = None
        self.verify_password_callback = None

    def verify_password(self, f):
        self.verify_password_callback = f
        return f
```

这里的`self.verify_password_callback`是回调函数，这里就需要我们手动写回调函数，然后将回调函数传入`verify_password`，然后再将函数回调给`verify_password_callback`。关于什么是回调函数，可以在我的博客中查看，链接地址：[点击这儿](https://www.dreamer.im/2019/04/16/%E9%9A%8F%E7%AC%94/%E7%AE%80%E5%8D%95%E4%BB%8B%E7%BB%8D%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0/)。

### 添加verify_password回调函数

关于支持令牌身份验证，我还需在User模型中定义两个新方法。

```python
#app/models.py 支持基于令牌的身份验证
class User(UserMixin,db.Model):
    #...
    def generate_auth_token(self,expiration):
        s = Serializer(current_app.config['SECRET_KEY'],
                       expires_in=expiration)
        return s.dumps({id}:self.id).decode('utf-8')
    
	@staticmethod
    def verify_auth_token(token):
        s = Serializer(current_app.config['SECRET_KEY'])
        try:
            data = s.loads(token)
        except:
            return None
        return User.query.get(data['id'])
```

下面需要我们添加`verify_password`回调函数，并且该函数支持基于令牌的身份验证。

```python
auth = HTTPBasicAuth()

#初始化Flask-HTTPAuth
@auth.verify_password
def verify_password(email_or_token,password):
    if email_or_token== '':
        return False
    if password=='':
        g.current_user = User.verify_auth_token(email_or_token)
        g.token_used = True
        return g.current_user is not None
    user = User.query.filter_by(email=email_or_token).first()
    if not user:
        return False
    g.current_user = user
    g.token_used = False
    return user.verify_password(password)
```

`generate_auth_token()`方法使用编码后的用户id字段值生成一个签名令牌，还制定了以秒为单位的过期时间。`verify_auth_token()`方法接受的参数式一个令牌，如果令牌有效就返回对于的用户。`verify_auth_token()`式静态方法，因为只有解码令牌后才能知道用户是谁。

为了能够使用令牌验证请求，我们必须修改`Flask-HTTPAuth`提供的`verify_password`回调，除了普通的凭据之外，还要接受令牌。

自此，身份验证模块还差最后一步，就是处理错误。当用户身份凭证不争取的时候，服务器需要向客户端返回401状态码。默认情况下，`Flask-HTTPAuth`自动生成这个状态码，但为了与API返回的其他状态码保持一致，我们可以自定义这个错误响应。

```python
#app/api/authentication.py Flask-HTTPAuth错误处理程序
from .errors import unauthorized

@auth.error_handler
def auth_error():
    return unauthorized('Invalid credentials')
```

```python
#app/api/errors.py API蓝本中各错误状态码的错误处理程序
from flask import jsonify
from app.exceptions import ValidationError
from . import api

def bad_request(message):
    response = jsonify({'error':'bad request','message':message})
    response.status_code = 400
    return response

def unauthorized(message):
    response = jsonify({'error':'unauthorized','message':message})
    response.status_code = 401
    return response

def forbidden(message):
    response = jsonify({'error':'forbidden','message':message})
    response.status_code = 403
    return response

@api.app_errorhandler(ValidationError)
def validation_error(e):
    return bad_request(e.args[0])
```

到此，身份验证模块已介绍完毕。当请求每个资源的时候，需要每次加上请求首部才能访问资源。

## 资源权限验证

我们可以利用之前写好的权限，在api蓝本中加上一个资源权限验证的装饰器`permission_required`。

```python
#app/api/decorators.py 资源权限验证装饰器
from functools import wraps
from flask import g
from .errors import forbidden

def permission_required(permission):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args,**kwargs):
            if not g.current_user.can(permission):
                return forbidden('Insufficient permissions')
            return f(*args,**kwargs)
        return decorated_function
    return decorator
```

在进行某些资源权限认证时加上`@permission.required`就可以了。例如：

```python
#创建一篇文章博客
@api.route('/posts/',methods=['POST'])
@permission_required(Permission.WRITE)
def new_post():
    post = Post.from_json(request.json)
    post.author = g.current_user
    db.session.add(post)
    db.session.commit()
    #返回201状态码，并把Location首部的值设为刚创建的这个资源的URL
    #为了方便客户端操作，相应的主体中包含了新建的资源
    return jsonify(post.to_json()), 201 ,{'Location':url_for('api.get_post',id=post.id)}
```

