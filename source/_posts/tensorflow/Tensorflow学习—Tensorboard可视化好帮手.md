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

当使用Tensorflow训练大量深层的神经网络时，我们希望去跟踪神经网络的整个训练过程中的信息，比如迭代的过程中每一层参数是如何变化与分布的，比如每次循环参数更新后模型在测试集与训练集上的准确率是如何的，比如损失值的变化情况，等等。如果能在训练的过程中将一些信息加以记录并可视化得表现出来，是不是对我们探索模型有更深的帮助与理解呢？

Tensorflow官方推出了可视化工具Tensorboard，可以帮助我们实现以上功能，它可以将模型训练过程中的各种数据汇总起来存在自定义的路径与日志文件中，然后在指定的web端可视化地展现这些信息。

## 1 Tensorboard介绍

### 1.1 Tensorboard的数据形式

Tensorboard可以记录与展示以下数据形式： 
（1）标量Scalars 
（2）图片Images 
（3）音频Audio 
（4）计算图Graph 
（5）数据分布Distribution 
（6）直方图Histograms 
（7）嵌入向量Embeddings

### 1.2 Tensorboard的可视化过程

（1）首先肯定是先建立一个graph,你想从这个graph中获取某些数据的信息

（2）确定要在graph中的哪些节点放置summary operations以记录信息 
	使用tf.summary.scalar记录标量 
	使用tf.summary.histogram记录数据的直方图 
	使用tf.summary.distribution记录数据的分布图 
	使用tf.summary.image记录图像数据 
	….

（3）operations并不会去真的执行计算，除非你告诉他们需要去run,或者它被其他的需要run的operation所依		赖。而我们上一步创建的这些summary operations其实并不被其他节点依赖，因此，我们需要特地去运行所有的summary节点。但是呢，一份程序下来可能有超多这样的summary 节点，要手动一个一个去启动自然是及其繁琐的，因此我们可以使用tf.summary.merge_all去将所有summary节点合并成一个节点，只要运行这个节点，就能产生所有我们之前设置的summary data。

（4）使用tf.summary.FileWriter将运行后输出的数据都保存到本地磁盘中

（5）运行整个程序，并在命令行输入运行tensorboard的指令，之后打开web端可查看可视化的结果

## 2 上一节简单神经网络关于tensorboard的使用案例

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

### 2.1 结果截图：

#### 2.1.1 loss的变化率

![loss](http://imgss.lovebingzi.com/tensorboard/loss.png)

#### 2.1.2 神经网络结构：

![graph](http://imgss.lovebingzi.com/tensorboard/graph.png)

点开还能查看各类参数

![](http://imgss.lovebingzi.com/tensorboard/graph2.png)



#### 2.1.3各参数数据变化：

![](http://imgss.lovebingzi.com/tensorboard/graphs.png)

