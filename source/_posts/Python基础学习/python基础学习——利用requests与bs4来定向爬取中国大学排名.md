title: python基础学习——利用requests与bs4来定向爬取中国大学排名
author: Imdreamer
tags:
  - python
  - requests
  - bs4
  - 爬虫
categories: []
date: 2018-02-02 21:31:00
---
python基础学习——利用requests与bs4来定向爬取中国大学排名
<!--more-->
{% codeblock lang:python%}
import requests
import bs4
from bs4 import BeautifulSoup

def getHTMLText(url):
    try:
        r = requests.get(url,timeout = 30)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return ""
    return ""
def fillUnivList(ulist,html):
    try:
        soup = BeautifulSoup(html,"html.parser")
        i=1
        for tr in soup.find('tbody').children:
            if isinstance(tr,bs4.element.Tag):
                tds = tr('td')
                ulist.append([i,tds[1].string,tds[3].string])
                i=i+1

    except:
        return ""
def printUnivList(ulist,num):
    try:
        tplt = "{0:^10}\t{1:{3}^10}\t{2:^10}"
        print(tplt.format("排名","学校名称","总分",chr(12288)))
        for i in range(num):
            u = ulist[i]
            print(tplt.format(u[0],u[1],u[2],chr(12288)))

        print("Suc"+str(num))
    except:
        return ""

if __name__ =="__main__":
    uinfo = []
    url = "http://www.zuihaodaxue.cn/zuihaodaxuepaiming2017.html"
    html = getHTMLText(url)

    fillUnivList(uinfo,html)
    printUnivList(uinfo,20)
{% endcodeblock %}