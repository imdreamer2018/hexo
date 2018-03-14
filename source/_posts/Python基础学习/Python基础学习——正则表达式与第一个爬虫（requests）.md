title: ' Python基础学习——正则表达式与第一个爬虫（requests）'
tags:
  - python
  - re
  - 爬虫
categories: []
date: 2018-02-02 14:55:00
---
关于第一个简单爬虫，利用requests库来实现
关于requests的安装：

安装很简单，我是win系统，就在这里下载了安装包（网页中download the zipball处链接），然后$ python setup.py install就装好了。
当然，有easy_install或pip的朋友可以直接使用：easy_install requests或者pip install requests来安装。
至于linux用户，这个页面还有其他安装方法。
测试：在IDLE中输入import requests，如果没提示错误，那说明已经安装成功了！
<!-- more -->
{% codeblock lang:python%}
import re
import urllib.request
import requests
#　urllib.urlopen()方法用于打开一个URL地址。
req = urllib.request.urlopen('https://www.imooc.com/')
#read()方法用于读取URL上的数据
buf = req.read().decode()
#decode bytes->str，encode str->bytes
#re.findall() 方法读取buf中包含（正则表达式）的数据,并形成一个list
listurl = re.findall(r'src=\"(.*?\.jpg)',buf)
i=0
for  url in listurl:
    f = open(str(i)+'.jpg','wb')
    url = 'http:' + url
    #获取文本
    data = requests.get(url).text
    #下载图片等二进制文件
    data = requests.get(url).content
    f.write(data)
    f.close()
    i+=1

{% endcodeblock %}
以下是关于正则表达式的一些用法	
![](/images/re.png)