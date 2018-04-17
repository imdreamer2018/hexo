title: Tensorflow学习—Tensorboard可视化好帮手
author: 追梦人
tags:
  - tensorflow
  - MachineLearning
  - tensorboard
categories: []
date: 2018-04-17 10:34:00
---

# 利用tensorboard来可视化显示

此教材来自[莫凡Python](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/)，本文旨在分享深度学习知识，如有侵权，可联系本人下架文章。

<!--more-->

```
import tensorflow as tf
import numpy as np
import matplotlib.pyplot as plt

#输入参数，行列矩阵维数，神经层级，激励函数
def add_layer(inputs,in_size,out_size,n_layer,activation_function=None): 
	#层级
    layer_name = 'layer%s'% n_layer
    with tf.name_scope(layer_name):
        with tf.name_scope('weights'):
        	#定义权重为行(in_zise)列(out_size)的随机矩阵，可视化中weights命名为W
            Weights = tf.Variable(tf.random_normal([in_size,out_size]),name='W') 
            #在tensorboard中显示Weighs
            tf.summary.histogram(layer_name+'/weights',Weights)
        with tf.name_scope('biases'):
            biases = tf.Variable(tf.zeros([1,out_size]) + 0.1,name='b')           #定于误差
            tf.summary.histogram(layer_name+'/biases',biases)
        with tf.name_scope('Wx_plus_b'):
            Wx_plus_b = tf.matmul(inputs,Weights) + biases                        #线性运算，W*x+biases
        if activation_function is None:
            outputs = Wx_plus_b
        else:
            outputs = activation_function(Wx_plus_b)
            tf.summary.histogram(layer_name+'/outputs',outputs)
        return outputs

x_data = np.linspace(-1,1,300)[:,np.newaxis]
noise = np.random.normal(0,0.05,x_data.shape) #添加噪点
y_data = np.square(x_data) -0.5 + noise      #spuare平方

#添加inputs可视化
with tf.name_scope('inputs'):
    xs = tf.placeholder(tf.float32,[None,1],name='x_input')     #输入x值
    ys = tf.placeholder(tf.float32,[None,1],name='y_input')     #输入y值
    
#两个隐藏层神经网络
l1 = add_layer(xs,1,10,n_layer=1,activation_function = tf.nn.relu) #通过神经层得到一个预测值
prediction = add_layer(l1,10,1,n_layer=2,activation_function=None)

#添加Loss可视化
with tf.name_scope('loss'):
    #得到误差
    loss = tf.reduce_mean(tf.reduce_sum(tf.square(ys-prediction),reduction_indices=[1])) 
    tf.summary.scalar('loss',loss)

#添加train可视化
with tf.name_scope('train'):
    train_step = tf.train.GradientDescentOptimizer(0.1).minimize(loss)  #通过线性回归（梯度下降）来减小Loss的值，学习率为0.1
    
    
sess = tf.Session()
#将所有视图整合一起
merged = tf.summary.merge_all()
#将视图信息写入当前目录logs/中
writer = tf.summary.FileWriter("logs/",sess.graph)
init = tf.initialize_all_variables()
sess.run(init)

for i in range(1000):
    sess.run(train_step,feed_dict={xs:x_data,ys:y_data})
    if i % 50 ==0:
        result = sess.run(merged,feed_dict={xs:x_data,ys:y_data})
        #每50个点显示一次坐标
        writer.add_summary(result,i)
```

输出结果后，在本地生成一个文件，如图

![logs](http://imgss.lovebingzi.com/tensorboard/logs.png)

最后进入终端，进入到这个目录下，用命令

```
tensorboard --logdir='logs/'
```

![](http://imgss.lovebingzi.com/tensorboard/tensorboard.png)

## 结果截图：

### loss的变化率

![loss](http://imgss.lovebingzi.com/tensorboard/loss.png)

### 神经网络结构：

![graph](http://imgss.lovebingzi.com/tensorboard/graph.png)

### 各参数数据变化：

![](http://imgss.lovebingzi.com/tensorboard/graphs.png)

