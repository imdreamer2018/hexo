title: Python Web 部署
author: 追梦人

toc: true

tags:

  - python
  - Flask
  - gunicorn
  - supervisor
  - nginx
categories: []
date: 2019-04-16 16:55:00

---

# Python Web 部署： 使用 flask + gunicorn + supervisor + nginx

本文转载自掘金，原文链接：[点击这儿](https://juejin.im/post/5a30f7f0f265da43346fe8b5)！

- **flask**   python 的服务器框架
- **gunicorn**   webservice,WSGI 的容器
- **supervisor**   进程管理工具
- **nginx**   一个高性能的 web 服务器

<!-- more -->

### 创建项目

```
mkdir server
复制代码
```

先为应用创建一个路径

### 构建 Python 的虚拟环境

我们使用 `virtualenv` 来构建一个系统中不同的 `python` 隔离环境, `virtualenv` 的使用十分的简单,**安装**和**使用方法**可以看这里[virtualenv](https://link.juejin.im?target=http%3A%2F%2Fpythonguidecn.readthedocs.io%2Fzh%2Flatest%2Fdev%2Fvirtualenvs.html)

```shel
cd server // cd 切换到我们的项目目录
virtualenv venv // 构建我们的虚拟环境
```

创建了 `venv` 环境后，我们需要激活才能使用(有时是自动激活)，激活后可以看见控制台前面有 `(venv)`

```shell
source venv/bin/activate
```

关闭环境直接使用 `deactivate`

```shell
deactivate
```

### 安装 flask 框架

安装的虚拟环境里面已经自动安装了 `pip`，我们使用 `pip` 可以很简单快捷的安装 `flask`

```she
pip install flask
```

flask 已经安装好了，我们可以通过一个小应用来测试一下我们的flask 框架, 'vim myapp.py' 创建一个 python 文件

```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return 'hello world !'

if __name__ = '__main__':
    app.debug = True
    app.run()
```

启动脚本

```
python myapp.py
```

此时，使用浏览器访问 [http://127.0.0.1:5000](https://link.juejin.im?target=http%3A%2F%2F127.0.0.1%3A5000) 就能看见网页显示的 **hello world !**

### 使用 gunicorn 部署 python web

刚才打开的是 `flask` 自带的服务器，完成了 web 服务的启动。但是 `flask` 自带的服务器一般是用来调试使用的，性能不佳。这里我们使用 `gunicorn` 作为 wsgi 的容器，用来部署 python。

##### 安装 gunicorn

```
pip install gunicorn
```

pip 是 python 的管理包工具。随着项目增长，你会发现它的依赖列表也一并随着增长。在你能运行一个Flask应用之前，即使已经需要数以十记的依赖包也毫不奇怪。 管理依赖的最简单的方法就是使用一个简单的文本文件。 pip可以生成一个文本文件，列出所有已经安装的包。它也可以解析这个文件，并在新的系统（或者新的环境）下安装每一个包。

```
pip freeze > requirements.txt # 生成txt 文件

pip install -r requirements.txt # 别人使用时可以直接安装所有的包
复制代码
```

以后每次 pip 安装了新的库的时候，都需 `freeze` 一次，保证更新

接下来我们就是用 `gunicorn` 来启动 `flask`

```
gunicorn -w 4 -b 0.0.0.0:8000 myapp:app
复制代码
```

此时我们使用8000端口进行访问，`-w` 表示开启了多少个 `worker`, `-b` 表示访问地址。`myapp` 就是 `myapp.py` 的文件名，`mypp.py` 相当于一个库文件被 `gunicorn` 调用。`app` 则是 `myapp.py` 里创建的 `app`，这样 `gunicorn` 才可以定位 `flask` 应用。 想结束 `gunicorn` 可以通过执行 `pkill gunicorn`，有时还要找到 `pid` 才能 kill 掉。这样的操作过于繁琐，所以我们使用另一个神器 `supervisor`, 用来专门管理系统的进程。

### 安装 supervisor

```
安装支持python3的supervisor进程管理工具
pip3 install git+https://github.com/Supervisor/supervisor
在项目根目录下创建conf文件夹用来保存supervisor相关配置信息
mkdir conf
将supervisor配置文件重定向到我们创建的文件夹下，方便管理
echo_supervisord_conf > supervisor.conf # 生成 supervisor 默认配置文件
vim supervisor.conf # 修改 supervisor 配置文件，添加 gunicorn 进程管理
复制代码
```

在 `supervisor.conf` 配置文件底部添加 (**注意我的工作路径是/var/www/server**)

```shell
[program:myapp]
command=/var/www/server/venv/bin/gunicorn -w4 -b0.0.0.0:2170 myapp:app  ; supervisor启动命令
directory=/var/www/server                                              ; 项目的文件夹路径
startsecs=0                                                            ; 启动时间
stopwaitsecs=0                                                         ; 终止等待时间
autostart=false                                                        ; 是否自动启动
autorestart=false                                                      ; 是否自动重启
stdout_logfile=/var/www/server/log/gunicorn.log                        ; log 日志
stderr_logfile=var/www/server/log/gunicorn.err  
```

其中的 log 目录是用来记录日志的，我们需要先创建一个 log 目录，否则会碰见未找到目录的错误

```shel
mkdir log
```

**supervisor 的基本使用命令**

```shell
supervisord -c supervisor.conf                             通过配置文件启动supervisor
supervisorctl -c supervisor.conf status                    察看supervisor的状态
supervisorctl -c supervisor.conf reload                    重新载入 配置文件
supervisorctl -c supervisor.conf start [all]|[appname]     启动指定/所有 supervisor管理的程序进程
supervisorctl -c supervisor.conf stop [all]|[appname]      关闭指定/所有 supervisor管理的程序进程
复制代码
```

### 部署 Nginx

`nginx` 是一个高性能的 `HTTP` 和 反向代理服务器，在高并发方面表现非常不错。

#### 安装 nginx

```
sudo apt-get install nginx
复制代码
```

nginx 安装完后，我们可以通过以下命令控制 nginx 的开启和关闭

```shell
sudo /etc/init.d/nginx restart // 重启
sudo /etc/init.d/nginx start 开启
sudo /etc/init.d/nginx stop 关闭
复制代码
```

**配置 nginx**

```shel
cd /etc/nginx/sites-available/default
cd /etc/nginx/sites-enabled/default
```

这是 nginx 的具体应用的配置文件，便于管理。修改默认的 default 文件

```nginx
server {
  #侦听80端口
    listen 80;
#定义使用www.xx.com访问
    server_name www.app.com; // 或则是地址(http://118.89.235.150/)
    client_max_body_size 10M;
 
   #设定本虚拟主机的访问日志
    access_log logs/app.log main;
 
  #默认请求
     location / {
        #请求转向本机ip:5000
        proxy_pass http://0.0.0.0:5000;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
  
    #配置静态文件转发
    location ^~/static/ {
        alias /www/wwwroot/yourwebsite/app/static/;
    }
    #配置静态文件转发
    location ~.*(js|css|png|gif|jpg|mp3|ogg)$ {
        root /home/zhoujianghai/temp/data/app/medias/;
    }
    #配置静态页面转发
    location ~.*(html)$ {
        root /home/zhoujianghai/temp/data/app/app_static_pages/;
    }
}
```

重启你的 nginx 就可以在浏览器上通过域名访问你的应用了。关于nginx的root和alias配置方法，可以参考这本[文章。](https://www.jianshu.com/p/4be0d5882ec5)