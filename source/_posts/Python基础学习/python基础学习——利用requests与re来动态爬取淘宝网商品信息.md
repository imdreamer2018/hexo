title: python基础学习——利用requests与re来动态爬取淘宝网商品信息
author: Imdreamer
tags:
  - python
  - requests
  - re
  - 爬虫
categories: []
date: 2018-02-03 17:05:00
---
python基础学习——利用requests与re来动态爬取淘宝网商品信息
<!--more-->

{%codeblock lang:python %}
import requests
import re

#获取页面信息
def getHTMLText(url):
    try:
        r = requests.get(url,timeout =30)
        r.raise_for_status()
        #转码
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return ""
#解析页面信息
def parsePage(ilt,html):
    try:
        #通过查看页面源代码可知，商品名称和价格是"raw_title":"**.**"和"view_price":"****",以下是正则表达式
        plt = re.findall(r'\"view_price\"\:\"[\d]*\.[\d]*\"',html)
        tlt = re.findall(r'\"raw_title\"\:\".*?\"',html)
        #通过循环逐个添加商品名称和价格到ilt，其中eval是出去字符串中的""，split(':'[1]是得到字符串中:画面的字符串)
        for i in range(len(plt)):
            price = eval(plt[i].split(':')[1])
            title = eval(tlt[i].split(':')[1])
            ilt.append([price,title])
    except:
        return ""

#打印列表信息
def printGoodsList(ilt):
    #规范打印格式
    tplt = "{:4}\t{:8}\t{:16}"
    print(tplt.format("序号","价格","商品名称"))
    count = 0
    for g in ilt:
        count = count+1
        print(tplt.format(count,g[0],g[1]))

if __name__ == "__main__":
    #商品名称
    goods = 'python'
    #爬取页面得深度
    depth = 3
    start_url = 'https://s.taobao.com/search?q=' +goods
    infoList =[]
    #循环获取页面信息并解析页面信息
    for i in range(depth):
        try:
            #淘宝得下一页是url + &s=44*i,下一页就是44，第三页就是88
            url = start_url + '&s=' + str(44*i)
            html = getHTMLText(url)
            parsePage(infoList,html)
        #如果本页信息解析失败，不会影响下一页的解析，继续作用函数
        except:
            continue
    #打印页面信息
    printGoodsList(infoList)

{%endcodeblock %}

<b>运行结果</b>
![upload successful](/images/pasted-0.png)