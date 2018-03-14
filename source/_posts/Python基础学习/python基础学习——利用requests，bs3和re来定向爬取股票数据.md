title: python基础学习——利用requests，bs4和re来定向爬取股票数据
author: Imdreamer
tags:
  - python
  - 爬虫
  - requests
  - re
categories: []
date: 2018-02-04 22:13:00
---
python基础学习——利用requests，bs4和re来定向爬取股票数据
<!--more-->
{% codeblock lang:python%}
import requests
import time
from bs4 import BeautifulSoup
import traceback
import re
#爬取页面信息
def getHTMLText(url,code = 'utf-8'):
    try:
        #为防止反爬虫，设置休眠时间
        #time.sleep(5)
        r = requests.get(url,timeout=30,)
        r.raise_for_status()
        r.encoding = code
        return r.text
    except:
        return ""
#解析页面信息，爬取东方财富网的各股票代码
def getStockList(lst,stockURL):
    html = getHTMLText(stockURL,'GB2312')
    soup = BeautifulSoup(html,'html.parser')
    a = soup.find_all('a')

    for i in a:
        try:
            href = i.attrs['href']
            lst.append(re.findall(r's[hz]\d{6}',href)[0])
        except:
            continue
#得到百度股票关于该股票的信息
def getStockInfo(lst,stockURL,fpath):
    count = 0
    #循环得到各个股票的信息
    for stock in lst:
        url = stockURL+stock+".html"
        html = getHTMLText(url)
        try:
            if html=='':
                continue
            #将股票信息存入字典infoDict中
            infoDict = {}
            soup = BeautifulSoup(html,'html.parser')
            stockInfo = soup.find('div',attrs={'class':'stock-bets'})
            #由于东方财富网上面的股票信息，在百度股票中不存在，所以会导致stockInfo为None
            if not stockInfo:
                continue
            #得到股票的名称
            name = stockInfo.find_all(attrs = {'class':'bets-name'})[0]
            #更新字典
            infoDict.update({'股票名称':name.text.split()[0]})
            #找到所有dt与dd的标签，dt存储股票信息key，dd标签存储value
            keyList = stockInfo.find_all('dt')
            valueList = stockInfo.find_all('dd')
            #由于有的股票已经休业，就得到不到各个信息，即为None
            if not keyList or not valueList:
                continue
            #循环存入字典中
            for i in range(len(keyList)):
                key = keyList[i].text
                val = valueList[i].text
                infoDict[key] = val
            #将字典写入本地文件中
            with open(fpath,'a+',encoding='utf-8') as f:
                f.write(str(infoDict)+'\n')
                count = count+1
                #为增加用户友好性交互，加上进度条
                print('\r当前速度:{:.2f}%'.format(count*100/len(lst)),end='')
        except:
            traceback.print_exc()
            count = count + 1
            print('\r当前速度:{:.2f}%'.format(count * 100 / len(lst)), end='')
            continue

if __name__=="__main__":
    stock_list_url = "http://quote.eastmoney.com/stocklist.html"
    stock_info_url = "https://gupiao.baidu.com/stock/"
    output_file = "D://BaiduStockInfo.txt"
    slist =[]
    getStockList(slist,stock_list_url)
    getStockInfo(slist,stock_info_url,output_file)

{% endcodeblock %}