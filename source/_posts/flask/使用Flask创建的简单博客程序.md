title: 使用Flask创建的简单博客程序
author: 追梦人

toc: true

tags:

  - python
  - Flask
  - gunicorn
  - supervisor
  - nginx
categories: []
date: 2020-05-16 16:55:00

---

# 使用Flask创建的简单博客程序

<p align="center">

<img align="center" src="http://imdreamer.oss-cn-hangzhou.aliyuncs.com/picGo/Flask_logo.svg"/>

<p align="center" style="font-size:30px;"><b>SimpleBlog</b></p>

------

<p align="center" style="color:grey;font-size:15px;">Simple blog web application by flask</p>

<!-- more -->

<img align="center" src="http://imdreamer.oss-cn-hangzhou.aliyuncs.com/picGo/QQ20200517-101553.png"/>

## Table of Contents

- [Background](https://github.com/imdreamer2018/SimpleBlog#background)
- [Install](https://github.com/imdreamer2018/SimpleBlog#install)
- [License](https://github.com/imdreamer2018/SimpleBlog#license)

## Background

<font face="roman">This blog web application build by flask when I learn flask web programming.And the code come from this book of "Flask Web programming".By the way,the web application html support by Jinja2.</font>

![](http://imdreamer.oss-cn-hangzhou.aliyuncs.com/picGo/O1CN01av5qc11CNduyRTl1H_!!0-item_pic.jpg_430x430q90.jpg)

------

<font face="roman" size=4>**Application characteristics including**:</font>

- Post blog and comment
- Migrate mysql support
- Email support
- Authentication
- Role permissions
- REST API
- **Perfect application structure**

------

<font face="roman" size=4>You can click this [link](https://www.dreamer.im/tags/Flask/) get more detail and code tutorial about the web application.</font>

## Install

```shell
git clone https://github.com/imdreamer2018/SimpleBlog.git
cd SimpleBlog
pip install -r app/requirements/dev.txt
```

**Tree**:

```shell
├── app
├── config.py
├── Dockerfile
├── LICENSE
├── README.md
├── docker-compose.yml
├── docker-environment.env
├── gunicorn.conf.py
├── main.py
```

**step1**:Open **.env file**,configure system environment variables

```shell
#.env file
FLASK_CONFIG='development'
#mysql host,name,user,password
DB_HOST='ip'
DB_NAME='****'
DB_USER='root'
DB_PASS='******'
#Flask SECRET KEY
SECRET_KEY='HelloWorld'
#Mail system
MAIL_SERVER='smtp.*****.com'
MAIL_USERNAME='*****'
MAIL_PASSWORD='*****'
#Mail port and MAIL_USE_SSL default is 465 and True
MAIL_PORT = 465
MAIL_USE_SSL = 'True'
#Mail system sender,same as MAIL_USERNAME
FLASKY_MAIL_SENDER='admin@****.com'
#Web application will set this email be admin when you register users
FLASKY_ADMIN='*****@gmail.com'
```

**step2**: **migrate mysql data** when you make sure the environment variables are configured correctly.If you have some error question,you can click this [link](https://www.dreamer.im/tags/Flask/)get more detail about it.

```shell
#open the Terminal
#migrate db
flask db init
flask db migrate
flask db upgrate
#insert data to role table
flask shell
from app import models
models.Role.insert_roles()
#Then you will find the application migrate some tables to your mysql due to the Flask-Migrate
```

**step3**: **flask run**

then this web application will be successful runing.And click http://127.0.0.1:5000 

### Docker depoly

<font face="roman" size=4>Open **docker-environment.env file**,configure system environment variables like step1.And you can open docker-compose.yml configure docker properties.</font>

```shell
docker-compose up
```

## License

[MIT](https://github.com/imdreamer2018/SimpleBlog/blob/master/LICENSE) @Imdreamer

