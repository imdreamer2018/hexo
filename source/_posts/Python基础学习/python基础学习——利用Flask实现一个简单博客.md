title: python基础学习——利用Flask实现一个简单博客
author: Imdreamer
tags:
  - python
  - flask
  - Python-web
categories: []
date: 2018-02-02 21:45:00
---
python基础学习——利用Flask实现一个简单博客
<!--more-->
<b>1,pythonweb.py</b>
{% codeblock lang:python%}
import pymysql

from flask import Flask, request, session, g, redirect, url_for, abort, render_template, flash

app = Flask(__name__)

app.config['SECRET_KEY'] = '123456'

#显示视图
@app.route('/')
def show_entries():

    cursor = conn.cursor()
    sql = "select title,text from entries  order by id desc "
    cursor.execute(sql)
    entries = [dict(title=row[0],text=row[1]) for row in cursor.fetchall()]
    return render_template('show_entries.html',entries=entries)


#添加视图
@app.route('/add',methods=['POST'])
def add_entry():
    if  session["logged_in"] ==True:
        cursor = conn.cursor()
        sql = "insert into  entries(title,text) values('%s','%s')" % (request.form['title'],request.form['text'])
        cursor.execute(sql)
        conn.commit()
        flash('New entry was successful posted')
        return redirect(url_for('show_entries'))
    else:
        flash('You are not login !')
        return redirect(url_for('login'))


@app.route('/login1')
def login1():
    return render_template("login.html")
#登陆与注销
@app.route('/login',methods=['post'])
def login():

    cursor = conn.cursor()
    sql = "select * from user where username='%s' and password = '%s' "%(request.form['username'], request.form['password'])
    cursor.execute(sql)
    rs = cursor.fetchall()
    if len(rs)!=1:
        error = 'Invalid username and password'
        return render_template('login.html', error=error)
    else:

        session["logged_in"] =True
        flash('You are logged in')
        return redirect(url_for('show_entries'))

@app.route('/logout')
def logout():
    session['logged_in']=False
    flash('You are logged out')
    return redirect(url_for('show_entries'))

conn = pymysql.Connect(
    host = '********',
    port = 3306,
    user = 'root',
    passwd = '**********',
    db = 'python'
)


if __name__=='__main__':
    app.run(debug =True)
{% endcodeblock %}

