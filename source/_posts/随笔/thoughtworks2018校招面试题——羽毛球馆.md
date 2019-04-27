title: thoughtworks2018校招面试题——羽毛球馆

toc: true

author: 追梦人
tags:

  - python
  - thoughtworks
categories: []
date: 2019-04-27 20:07:00

---

# **2018校招面试题——羽毛球馆**

## **说明**

- 本作业限时2天完成

- 可以选用擅长的语言完成，例如C、C++、Java、C#、Javascript、Python、Scala等

- 可以使用第三方库简化代码（如日期，时间、集合操作等）

- 作业的输入和输出必须和题目的测试用例输出严格一致

- 作业完成后必须附上 Readme 纯文本文档（推荐使用 markdown 排版）

- Readme文档中应描述如何运行单元测试或主程序来证明作业的正确性（至少针对测试用例输入能够得到对应输出）

  <!-- more -->

## **家庭作业部分**

小明是一个羽毛球场馆的管理员，管理着四个羽毛球场地（A，B，C，D场地），负责场地的维护和预订工作。为了简化自己的工作，场地只接受整点预订，预订以小时为单位。

羽毛球场的收费标准如下：

- 周一到周五：
  - 9:00 ~ 12:00 30元/时
  - 12:00 ~ 18:00 50元/时
  - 18:00 ~ 20:00 80元/时
  - 20:00 ~ 22:00 60元/时
- 周六及周日
  - 9:00 ~ 12:00 40元/时
  - 12:00 ~ 18:00 50元/时
  - 18:00 ~ 22:00 60元/时

羽毛球场馆在预订之后，可以接受取消预订，不过取消预订需要交违约金，违约金的计算规则如下：

- 周一到周五的预订取消收取全部费用的50%作为违约金
- 周六周日的预订取消收取全部费用的25%作为违约金

由于手头还有其他工作，小明希望能够借助计算机程序来自动化处理预订及取消预订的事务，并且希望程序能够打印出场馆的收入汇总情况。

## **程序输入**

**预订：用户预订以字符串的形式输入，一行字符串代表一个预定**

- 格式为{用户ID} {预订日期 yyyy-MM-dd} {预订时间段 HH:mm~HH:mm} {场地}，如U123 2016-06-02 20:00~22:00 A，代表用户U123预定2016年06月02日晚上20:00到22:00的场地A的时间段
- 时间段的起止时间必然为整小时，否则报错
- 如预订与已有预订冲突，也会报错

**取消预定：用户取消预定，输入也以一行字符串的形式表现**

- 格式为{用户ID} {预订日期 yyyy-MM-dd} {预订时间段 HH:mm~HH:mm} {场地} {取消标记}，如U123 2016-06-02 20:00~22:00 A C，代表用户U123取消其在2016年06月02日晚上20:00到22:00在场地A的预订，其中取消标记C代表Cancel
- 取消标记只能是C，若为其他字符则报错
- 时间段的起止时间必然为整小时，否则报错
- 只能完整取消之前的预订，不能取消部分时间段
- 取消预订的请求，必须与之前的预订请求严格匹配，需要匹配的项有用户ID，预订日期，预订时间段，场地

**打印场馆收入汇总： 将所有的预订和取消预订带来的收入汇总信息打印出来**

- 格式为，输入一个空行，代表打印收入汇总

## **程序输出**

**收入汇总：以文本的形式输出当前系统所有预订以及取消预订所带来的收入情况，以不同的场地分组，一个可能的输出如下所示：**

```shell
收入汇总
---
场地:A
2016-06-02 09:00~10:00 违约金 15元
2016-06-02 10:00~12:00 60元
2016-06-03 20:00~22:00 120元
小计：195元
 
场地:B
2016-06-04 09:00~10:00 40元
小计：40元
 
场地:C
小计：0元
 
场地:D
小计：0元
---
总计: 235元
```

注意：

- 如果同一场地同一时间段有多条预定记录，则显示多条
- 收入记录以时间顺序升序排列

### **测试用例1**

注意：>开头表示命令行输出，以下测试用例都遵循此例

```shell
> abcdefghijklmnopqrst1234567890
> Error: the booking is invalid!
U001 2016-06-02 22:00~22:00 A
> Error: the booking is invalid!
U002 2017-08-01 19:00~22:00 A
> Success: the booking is accepted!
U003 2017-08-02 13:00~17:00 B
> Success: the booking is accepted!
U004 2017-08-03 15:00~16:00 C
> Success: the booking is accepted!
U005 2017-08-05 09:00~11:00 D
> Success: the booking is accepted!
 
> 收入汇总
> ---
> 场地:A
> 2017-08-01 19:00~22:00 200元
> 小计：200元
>
> 场地:B
> 2017-08-02 13:00~17:00 200元
> 小计：200元
>
> 场地:C
> 2017-08-03 15:00~16:00 50元
> 小计：50元
>
> 场地:D
> 2017-08-05 09:00~11:00 80元
> 小计：80元
> ---
> 总计：530元
```

### **测试用例2**

```shell
U002 2017-08-01 19:00~22:00 A
> Success: the booking is accepted!
U003 2017-08-01 18:00~20:00 A
> Error: the booking conflicts with existing bookings!
U002 2017-08-01 19:00~22:00 A C
> Success: the booking is accepted!
U002 2017-08-01 19:00~22:00 A C
> Error: the booking being cancelled does not exist!
U003 2017-08-01 18:00~20:00 A
> Success: the booking is accepted!
U003 2017-08-02 13:00~17:00 B
> Success: the booking is accepted!
 
> 收入汇总
> ---
> 场地:A
> 2017-08-01 18:00~20:00 160元
> 2017-08-01 19:00~22:00 违约金 100元
> 小计：260元
>
> 场地:B
> 2017-08-02 13:00~17:00 200元
> 小计：200元
>
> 场地:C
> 小计：0元
>
> 场地:D
> 小计：0元
> ---
> 总计：460元
```

## **办公室面试部分**

现在，羽毛球场推出不定期的优惠活动。活动期间，价格在总价上打相应折扣（折扣四舍五入到元）。管理员小明希望能够动态更新优惠信息，以更优惠的价格来服务到场馆运动的羽毛球爱好者。

球场的优惠活动存放在程序的某文本资源文件中，其中的每一行字符串代表一个优惠时间段，每个优惠时段的格式为，{起始日期：yyyy-MM-dd} {终止日期（包含）：yyyy-MM-dd} {折扣：1-9代表相应的折扣}，以下为范例:

```shell
2016-04-01 2016-04-02 6
2017-08-01 2017-08-03 8
```

请修改程序实现优惠后的收入汇总，需要注意之前的所有需求保持不变。

### **测试用例**

注意：以下结果基于上述范例优惠信息计算得出

```shell
abcdefghijklmnopqrst1234567890
> Error: the booking is invalid!
U001 2016-06-02 22:00~22:00 A
> Error: the booking is invalid!
U002 2017-08-01 19:00~22:00 A
> Success: the booking is accepted!
U003 2017-08-02 13:00~17:00 B
> Success: the booking is accepted!
U004 2017-08-03 15:00~16:00 C
> Success: the booking is accepted!
U005 2017-08-05 09:00~11:00 D
> Success: the booking is accepted!
 
> 收入汇总
> ---
> 场地:A
> 2017-08-01 19:00~22:00 160元 已优惠:40元
> 小计：160元
>
> 场地:B
> 2017-08-02 13:00~17:00 160元 已优惠:40元
> 小计：160元
>
> 场地:C
> 2017-08-03 15:00~16:00 40元 已优惠:10元
> 小计：40元
>
> 场地:D
> 2017-08-05 09:00~11:00 80元
> 小计：80元
> ---
> 总计：440元
```

## 项目需求分析

从上述题目说明来看，目前是需要设计一个系统，用于帮助记录羽毛球馆租赁情况，其中主要功能如下：

1. 预定场地
2. 取消场地
3. 汇总信息

另外还有一些规范：

1. 程序输入格式
2. 主要功能逻辑上的处理

## 设计

针对上述需求分析，系统设计流程图如下所示。

![EK7kJe.png](https://s2.ax1x.com/2019/04/27/EK7kJe.png)

### **代码文件结构**

```
#代码文件结构
|-thoughtworks
   |-app.py
   |-CancelControler.py
   |-ChargesControler.py
   |-config.py
   |-DataControler.py
   |-InsertControler.py
   |-ModelControler.py
   |-OrderControler.py
   |-test.client.py
|-venv
```

### 各类模块说明

| 模块             | 文件名              | 说明                                                         |
| ---------------- | ------------------- | ------------------------------------------------------------ |
| 程序入口模块     | app.py              | 整个系统的程序入口文件                                       |
| 程序设置模块     | Config.py           | 系统的初始设置文件，包括营业时间以及收费标准、违约金比例、开放场地、打折时间等。 |
| 公共计费标准模块 | ChargesControler.py | 系统的计费函数，输入时间，输出收费金额等。                   |
| 公共数据存储模块 | DataControler.py    | 系统的数据存储函数，为了简便，这里主要是采用字典的形式存储数据，程序运行结束后就会重置。可以考虑采用数据库的形式。 |
| 输入控制模块     | InsertControler.py  | 系统的输入控制函数，主要是为了检验输入是否合法，拦截垃圾输入。 |
| 功能模块控制     | ModelControler.py   | 功能模块控制，根据过滤后的输入信息，核查输入对应的功能，是预定、取消还是汇总功能。 |
| 预定功能模块     | OrderControler.py   | 预定场地功能的函数，并且能够检验场地是否被占用，如若通过检验，预定场地成功并将信息存入数据库，否则输出错误消息。 |
| 取消功能跟模块   | CancelControler.py  | 取消场地功能的函数，并且能够检验场地是否被预定，如若通过检验，取消场地成功并收取违约金，否则输出错误消息。 |
| 测试模块         | test_client.py      | 自动化测试系统，检验系统各个功能是否存在问题。               |

## 编码

### **程序入口模块**

```python
#thoughtworks/app.py
import sys
from thoughtworks.InsertControler import InsertControler
from thoughtworks import ModelControler

if __name__ == '__main__':
    while True:
    	#读取输入字符串
        str = sys.stdin.readline().strip()
        list = str.split(' ')
        result = InsertControler(list)
        if result:
        	#统计汇总
            if result == 1:
                ModelControler.total()
            #预定场地
            elif result == 2:
                ModelControler.order(list)
            #取消场地
            elif result == 3:
                ModelControler.cancel(list)
        else:
            print('Error: the booking is invalid!')
```

### **程序设置模块**

```python
#thoughtworks/Config.py
class Charges:
    #注册这里要按时间顺序来写，不可倒叙或者乱序，
    #列表依次为开始时间，结束时间，收费标准/每小时
    daily =[
        [9,12,30],
        [12,18,50],
        [18,20,80],
        [20,22,60]
    ]
    weekend =[
        [9,12,40],
        [12,18,50],
        [18,22,60]
    ]
    #违约金比例，列表依次为周一到周五，周六周日收取违约金比例
    breakPromise = [0.5,0.25]
    #场地
    site = ['A','B','C','D']
    #优惠活动时间，列表依次为开始时间，结束时间和优惠折扣
    Promotions = ['2016-04-01 2016-04-02 6','2017-08-01 2017-08-03 8']
```

### 公共计费模块

```python
#thoughtworks/ChargesControler.py
import time,datetime
from interval import Interval
from thoughtworks.Config import Charges

#时间转换为时间戳
def Time_Conversion(time_):
    timeList = time_.split(' ')
    day = timeList[0]
    timeArray = timeList[1].split('~')
    startTime = int(time.mktime(datetime.datetime.strptime(day +' '+ timeArray[0], '%Y-%m-%d %H:%M').timetuple()))
    endTime = int(time.mktime(datetime.datetime.strptime(day +' '+ timeArray[1], '%Y-%m-%d %H:%M').timetuple()))
    return (startTime,endTime)

#查询是否有优惠
def Promotions(day):
    day = int(time.mktime(datetime.datetime.strptime(day,'%Y-%m-%d').timetuple()))
    for i in Charges.Promotions:
        try:
            pro = i.split(' ')
            startday = int(time.mktime(datetime.datetime.strptime(pro[0],'%Y-%m-%d').timetuple()))
            endday = int(time.mktime(datetime.datetime.strptime(pro[1],'%Y-%m-%d').timetuple()))
            promotion = int(pro[2])
            if day >= startday and day <= endday:
                return promotion/10
        except:
            return False
    return 1

#计费
def ChargesControler(time):
    timeList = time.split(' ')
    day = timeList[0]
    promotion = Promotions(day)
    if not promotion:
        return 0, 0, 0, 0, 0, 0, 0
    timeArray = timeList[1].split('~')
    startTime = int(datetime.datetime.strptime(timeArray[0],'%H:%M').strftime('%H'))     #起始时间
    endTime = int(datetime.datetime.strptime(timeArray[1],'%H:%M').strftime('%H'))       #结束时间
    if endTime <= startTime:
        return 0, 0, 0, 0, 0, 0, 0
    whatday = int(datetime.datetime.strptime(str(timeList[0]),'%Y-%m-%d').strftime('%w'))#周几
    timeLength = 0                                                                       #时常
    cost = 0                                                                             #费用
    startTime_=startTime
    #周一~周五
    if whatday >= 1 and whatday <= 5:
        f,g = 0,0
        for i in Charges.daily:
            f += 1
            if startTime in Interval(i[0],i[1],upper_closed=False) and \
            	endTime in Interval(i[0],i[1],upper_closed=False):
                timeLength += (endTime-startTime)
                cost += (endTime-startTime) * i[2] * promotion
                break
            if startTime in Interval(i[0],i[1],upper_closed=False) and \
            	not endTime in Interval(i[0],i[1],upper_closed=False):
                timeLength += (i[1] - startTime)
                cost += (i[1] - startTime) * i[2] * promotion
                startTime = i[1]
            elif not startTime in Interval(i[0],i[1],upper_closed=False) and \
            	not endTime in Interval(i[0],i[1],upper_closed=False):
                g += 1
        if f == g:
            print('Error: the booking is invalid!')
            return 0, 0, 0, 0, 0, 0, 0
    #周六周日
    else:
        f, g = 0, 0
        for i in Charges.weekend:
            f += 1
            if startTime in Interval(i[0], i[1], upper_closed=False) and \
          	  endTime in Interval(i[0],i[1], upper_closed=False):
                timeLength += (endTime - startTime)
                cost += (endTime - startTime) * i[2] * promotion
                break
            if startTime in Interval(i[0], i[1], upper_closed=False) and \
            	not endTime in Interval(i[0],i[1], upper_closed=False):
                timeLength += (i[1] - startTime)
                cost += (i[1] - startTime) * i[2] * promotion
                startTime = i[1]
            elif not startTime in Interval(i[0],i[1],upper_closed=False) and \
           		not endTime in Interval(i[0],i[1],upper_closed=False):
                g += 1
        if f == g:
            print('Error: the booking is invalid!')
            return 0, 0, 0, 0, 0, 0, 0
    return 1,whatday,startTime_,endTime,timeLength,cost,promotion
```

### 公共数据存储模块

```python
#thoughtworks/DataControler.py
from datetime import datetime
from thoughtworks.ChargesControler import Time_Conversion
from thoughtworks.Config import Charges
import time

#数据库
class database:
    data = {
        'A':[],
        'B':[],
        'C':[],
        'D':[]
    }

#预定
def OrderDataControler(site,userID,time_,whatday,timeLength,sign,cost,promotion):
    startTime, endTime = Time_Conversion(time_)
    #数据存储格式。注：这里有一些不必要的数据可以删除。
    #用户ID，时间，开始和结束时间（小时），周几，时长，标识（1为预定中，0为已取消），收费金额，折扣
    data = {'userID':userID,
             'time':time_,
             'startTime':startTime,
             'endTime':endTime,
             'whatday':whatday,
             'timeLength':timeLength,
             'sign':sign,
             'cost':cost,
             'promotion':promotion
             }
    if site in database.data:
        try:
            #存入数据
            database.data[site].append(data)
        except:
            print('save error')
    else:
        print(' Error:the booking is invalid!')

#取消
def CancelDataControler(site,f):
    whatday = database.data[site][f]['whatday']
    cost = database.data[site][f]['cost']
    if whatday >= 1 and whatday <= 5:
        database.data[site][f]['sign'] = 0
        database.data[site][f]['cost']= cost*Charges.breakPromise[0]
    else:
        database.data[site][f]['sign'] = 0
        database.data[site][f]['cost'] = cost * Charges.breakPromise[1]
```

### 输入控制模块

```python
#thoughtworks/InsertControler.py
import time
import re
from thoughtworks.Config import  Charges

#检验用户名是否有效，匹配规则：U+若干位【0-9】数字
def is_valid_user(struser):
    if re.match(r'U\d+?', struser):
        return True
    else:
        return False

#检验日期字符串是否有效
def is_valid_date(strdate):
    '''判断是否是一个有效的日期字符串'''
    try:
        if ":" in strdate:
            time.strptime(strdate, "%H:%M")
        else:
            time.strptime(strdate, "%Y-%m-%d")
        return True
    except:
        return False

#输入控制
def InsertControler(list):
    #当为空格时，汇总信息
    if len(list)==1 and list[0] == '':
        return 1
    #预定输入时
    elif len(list) == 4 or len(list) == 5:
        #判断是否为取消预定
        if len(list) == 5 and list[4] != 'C':
            return False
        f = 0
        #判断场地是否为A,B,C,D
        for i in Charges.site:
            if  not list[3] is i:
                f += 1
        if f == 4:
            return False
        #判断用户名是否正确
        if not is_valid_user(list[0]):
            return False
        #判断年月日是否正确
        if not is_valid_date(list[1]):
            return False
        #判断时间是否正确并且为整点
        timelist = list[2].split('~')
        if len(timelist) == 2:
            if is_valid_date(timelist[0]) and is_valid_date(timelist[1]):
                time1 = timelist[0].split(':')
                time2 = timelist[1].split(':')
                if time1[1] != '00' or time2[1] != '00' or time1[0] >= time2[0]:
                    return False
            else:
                return False
        else:
            return False
        if len(list) == 4:
            return 2
        elif len(list) == 5:
            return 3
    else:
        return False
```

### 功能模块控制

```python
#thoughtworks/ModelControler.py
from thoughtworks.DataControler import database
from thoughtworks.OrderControler import OrderControler
from thoughtworks.CancelControler import CancelControler

#汇总
def total():
    print('收入汇总')
    print('--------')
    TotalCost = 0
    for i in database.data:
        total = 0
        print('场地：',i)
        for j in database.data[i]:
            if j['sign'] == 1 and j['promotion'] == 1:
                total += j['cost']
                print(j['time'],j['cost'],'元')
            elif j['sign'] == 1 and j['promotion'] != 1:
                total += j['cost']
                pro = j['cost'] / j['promotion'] - j['cost']
                print(j['time'], j['cost'], '元','已优惠',pro,'元')
            elif j['sign'] == 0:
                total += j['cost']
                print(j['time'],'违约金',j['cost'],'元')
        TotalCost += total
        print('小计',total,'元')
        print('--------')
    print('总计',TotalCost,'元')

#预定
def order(list):
    site = list[3]
    userID = list[0]
    time_ =  list[1] + ' ' + list[2]
    return OrderControler(site,userID,time_)

#取消
def cancel(list):
    site = list[3]
    userID = list[0]
    time_ = list[1] + ' ' + list[2]
    return CancelControler(site, userID, time_)
```

### 预定功能模块

```python
#thoughtworks/OrderControler.py
from thoughtworks.DataControler import database
from thoughtworks.DataControler import OrderDataControler
from thoughtworks.ChargesControler import ChargesControler
from thoughtworks.ChargesControler import Time_Conversion

#判断数据中是否存在时间段与预定时间段有交集
def Judge_Data_Exist(site,time_):
    startTime, endTime = Time_Conversion(time_)
    if database.data[site] is not None:
        for i in database.data[site]:
            if int(i['startTime']) < endTime and startTime < int(i['endTime']) \
            	and int(i['sign']):
                return False
    return True

#预定场地
def OrderControler(site,userID,time_):
    if Judge_Data_Exist(site,time_):
        f ,whatday, startTime_, endTime_, timeLength, cost, promotion = ChargesControler(time_)
        if f:
            sign =1
            try:
                OrderDataControler(site, userID, time_, whatday,
                                   timeLength,sign,cost,promotion)
                print('Success: the booking is accepted!')
                return True
            except:
                print('order error')
                return False
        else:
            print('Error:the booking is invalid!')
            return False
    else:
        print('Error: the booking conflicts with existing bookings!')
        return False
```

### 取消功能模块

```python
#thoughtworks/CancelControler.py
from thoughtworks.DataControler import database
from thoughtworks.ChargesControler import Time_Conversion
from thoughtworks.DataControler import CancelDataControler

#判断数据中是否存在该时间段可以取消
def Judge_Data_Exist(site,userID,time_):
    startTime, endTime = Time_Conversion(time_)
    f = 1
    for i in database.data[site]:
        if i['userID']== userID and int(i['startTime']) == startTime and endTime == int(i['endTime']) and i['sign']:
            return f
        f+=1
    return False

#取消
def CancelControler(site,userID,time_):
    f = Judge_Data_Exist(site,userID,time_)
    if f:
        try:
            CancelDataControler(site,f-1)
            print('Success: the booking is accepted!')
            return True
        except:
            print('cancel error')
            return False
    else:
        print('Error: the booking being cancelled does not exist!')
        return False
```

### 测试模块

```python
#thoughtworks/test_client.py
import unittest
from HTMLTestRunner import BSTestRunner
from thoughtworks.InsertControler import InsertControler
from thoughtworks.ModelControler import order,cancel,total

class TestMyApp(unittest.TestCase):

    @classmethod
    def setUpClass(cls):
        print('初始化环境')

    @classmethod
    def tearDownClass(cls):
        print('运行结束')
	
    #打印输入
    @staticmethod
    def test_print(str):
        print(str)
        return str
	
    #打印输出
    @staticmethod
    def test_validate(s):
        if s:
            print('Success: the booking is accepted!')
            return True
        else:
            print('Error: the booking is invalid!')
            return False
	
    #程序输入检验
    def test_InertControler(self):

        self.assertFalse(self.test_validate(InsertControler(
            self.test_print('abcdefghijklmnopqrst1234567890').split(' '))))
        self.assertFalse(self.test_validate(InsertControler(
            self.test_print('123213').split(' '))))
        self.assertFalse(self.test_validate(InsertControler(
            self.test_print('U001 2016-06-02 23:00~22:00 A').split(' '))))
        self.assertFalse(self.test_validate(InsertControler(
            self.test_print('U0012016-06-02 23:00~22:00 A').split(' '))))
        self.assertFalse(self.test_validate(InsertControler(
            self.test_print('U0012016-06-0223:00~22:00 A').split(' '))))
        self.assertFalse(self.test_validate(InsertControler(
            self.test_print('U001 2016-06-02 22:00~22:00 A').split(' '))))
        self.assertTrue(self.test_validate(InsertControler(
            self.test_print('U002 2017-08-01 19:00~22:00 A').split(' '))))
        self.assertTrue(self.test_validate(InsertControler(
            self.test_print('U003 2017-08-02 13:00~17:00 B').split(' '))))
        self.assertTrue(self.test_validate(InsertControler(
            self.test_print('U004 2017-08-03 15:00~16:00 C').split(' '))))
        self.assertTrue(self.test_validate(InsertControler(
            self.test_print('U005 2017-08-05 09:00~11:00 D').split(' '))))
	
	#预定场地检验
    def test_order(self):
        self.assertTrue(order(self.test_print(
            'U002 2017-08-01 19:00~22:00 A').split(' ')))
        self.assertFalse(order(self.test_print(
            'U003 2017-08-01 18:00~20:00 A').split(' ')))
        self.assertTrue(order(self.test_print(
            'U003 2017-08-01 13:00~19:00 A').split(' ')))
        self.assertTrue(order(self.test_print(
            'U003 2017-08-02 13:00~17:00 B').split(' ')))
        self.assertTrue(order(self.test_print(
            'U004 2017-08-03 15:00~16:00 C').split(' ')))
        self.assertTrue(order(self.test_print(
            'U005 2017-08-05 09:00~11:00 D').split(' ')))
	
    #取消场地检验
    def test_cancel(self):
        self.assertTrue(cancel(self.test_print(
            'U002 2017-08-01 19:00~22:00 A C').split(' ')))
        self.assertFalse(cancel(self.test_print(
            'U002 2017-08-01 19:00~22:00 A C').split(' ')))
	
    #统计汇总检验
    def test_total(self):
        total()


if __name__ == '__main__':
    tests = [TestMyApp('test_InertControler'),TestMyApp('test_order'),
             TestMyApp('test_cancel'),TestMyApp('test_total')]
    suite = unittest.TestSuite()
    suite.addTests(tests)
    #保存到本地为HTML格式报告
    with open('HTMLReport.html', 'wb') as f:
        runner = BSTestRunner.BSTestRunner(stream=f,
                    title='Result Report',
            		description='This demonstrates the report output by BSTestRunner.',
                    verbosity=2)
        runner.STYLESHEET_TMPL = '<link rel="stylesheet" href="my_stylesheet.css" type="text/css">'
        runner.run(suite)
```

## 测试

这次采用了`python`的`unittest`单元测试模块，参考的教程链接为：[点击这儿](https://blog.csdn.net/huilan_same/article/details/52944782)和[点击这儿](https://blog.51cto.com/2681882/2123613)。

具体教程说明可以参考上述的两个原文链接。最后均能通过单元测试。可能有一些地方存在Bug，但是目前还未发现。

![EKvman.gif](https://s2.ax1x.com/2019/04/27/EKvman.gif)