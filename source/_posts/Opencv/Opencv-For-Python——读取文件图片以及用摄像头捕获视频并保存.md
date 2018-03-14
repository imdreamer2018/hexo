title: Opencv For Python——读取文件图片以及用摄像头捕获视频并保存
author: Imdreamer
tags:
  - python
  - opencv
categories: []
date: 2018-02-24 15:03:00
---
Opencv For Python——读取文件图片以及用摄像头捕获视频并保存
<!--more-->

1,读取文件图片，并根据键盘输入是否保存
{% codeblock lang:python%}
  import cv2
import numpy as np

#读取图片
img = cv2.imread('test.jpg',0)
#自定义调整窗口大小
cv2.namedWindow('image',cv2.WINDOW_NORMAL)
#显示图像
cv2.imshow('image',img)
#等待键盘输入，为毫秒级
k = cv2.waitKey(0)
#当键盘输入esc时，图片不保存退出
if k==27:
    cv2.destroyAllWindows()
#当键盘输入s时，图片保存为1.png并退出
elif k==ord('s'):
    cv2.imwrite('1.png',img)
    cv2.destroyAllWindows()

{% endcodeblock %}
2,用摄像头捕获视频并输出保存再本地
{% codeblock lang:python%}
import numpy as np
import cv2

#创建一个VideoCapture对象，他的参数可以是设备的索引号或者一个视频文件
#设备索引号0->笔记本电脑的内置摄像头，也可以通过设置成1或者其他来选择别的摄像头
#从文件中播放视频，就是把设备索引号改成视频文件的名字就行
cap =cv2.VideoCapture(0)
#捕获视频并保存，需要创建一个VideoWriter
#FourCC就是一个4字节码，用来确定视频的编码格式
#In Fedora: DIVX,XVID,MJPG,X264,WMV1,WMV2(XVID是最适合的格式)
#In Windows: DIVX
fourcc = cv2.VideoWriter_fourcc(*'DIVX')
#VideoWriter（输出的文件名，FourCC编码，播放频率，帧的大小，isColor）当isColor为True时，每一帧时彩色，默认灰度
out = cv2.VideoWriter('output.avi',fourcc,20.0,(640,480))
#有时候cap可能不成功的初始化摄像头设备，这种情况代码回报错，这时我们用cap.isOpened(),来检查是否成功初始化
while(cap.isOpened()):
    ret,frame = cap.read()
    if ret ==True:
        #cv2.flip图像翻转
        # 1	  水平翻转   0	垂直翻转   -1	水平垂直翻转
        frame = cv2.flip(frame,0)
        #写入捕获的视频文件到本地
        out.write(frame)
        cv2.imshow('frame',frame)
        if cv2.waitKey(1) & 0xFF ==ord('q'):
            break
    else:
        break

cap.release()
out.release()
cv2.destroyAllWindows()
{% endcodeblock %}
