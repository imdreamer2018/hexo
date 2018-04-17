title: Tensorflow学习—构建一个简单的神经网络

author: 追梦人

toc: true

categories: []

date: 2018-4-16 21:36:00

tags:

- MachineLearning
- tensorflow

---

# 基于线性回归构建神经网络

此教材来自[莫凡Python](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/)，本文旨在分享深度学习知识，如有侵权，可联系本人下架文章。

<!--more-->

```
import tensorflow as tf
import numpy as np
import matplotlib.pyplot as plt
#定义隐藏层(输入参数，行列矩阵维数，激励函数)
def add_layer(inputs,in_size,out_size,activation_function=None): 
    #定义权重为行(in_zise)(out_size)的随机矩阵
    Weights = tf.Variable(tf.random_normal([in_size,out_size]))  
    biases = tf.Variable(tf.zeros([1,out_size]) + 0.1)           #定于误差
    Wx_plus_b = tf.matmul(inputs,Weights) + biases               #线性运算，W*x+biases
    if activation_function is None:
        outputs = Wx_plus_b
    else:
        outputs = activation_function(Wx_plus_b)
    return outputs

#定义线性函数实际数据
x_data = np.linspace(-1,1,300)[:,np.newaxis]
#对于numpy.random.normal函数，有三个参数（loc, scale, size），分别l代表生成的高斯分布的随机数的均值、方差以及输出的size.
noise = np.random.normal(0,0.05,x_data.shape) 					#添加噪点
y_data = np.square(x_data) -0.5 + noise     					#spuare平方

#定义xs,ys传入x_data,y_data值
xs = tf.placeholder(tf.float32,[None,1])    					#输入x值
ys = tf.placeholder(tf.float32,[None,1])     					#输入y值

#xs的随机值通过隐藏层，得到预测值
l1 = add_layer(xs,1,10,activation_function = tf.nn.relu) 		#通过神经层得到一个预测值
prediction = add_layer(l1,10,1,activation_function=None)

#计算实际值ys与prediction的误差
loss = tf.reduce_mean(tf.reduce_sum(tf.square(ys-prediction),reduction_indices=[1])) #得到误差
#通过线性回归（梯度下降）来减小Loss的值，学习率为0.1
train_step = tf.train.GradientDescentOptimizer(0.1).minimize(loss)  

#运行sess
init = tf.initialize_all_variables()				#当tensorflow定义变量时，必须加上这句代码
sess = tf.Session()
sess.run(init)

#定义一个绘图变量plt
fig = plt.figure()
#新增一个图层
ax = fig.add_subplot(1,1,1)
#通过散点的方式显示x_data与y_data的图形关系
ax.scatter(x_data,y_data)
#在Python3中可以用plt.ion()来动态显示图片变化
plt.ion()
#plt.show()

for i in range(1000):
	#以字典的方式来将x_data,y_data的值传入xs,ys，并运行train_step
    sess.run(train_step,feed_dict={xs:x_data,ys:y_data})
    if i % 50 ==0:
        print(sess.run(loss,feed_dict={xs:x_data,ys:y_data}))
    #由于每次显示预测值的连线，如果不除去上一次的连线，整个图形就会混乱，第一次没有连线，用try来屏蔽报错
        try:
             ax.lines.remove(lines[0])
        except Exception:
            pass
        prediction_value = sess.run(prediction,feed_dict={xs:x_data})
        #将预测值的点连成一条红线
        lines = ax.plot(x_data,prediction_value,'r-',lw=5)  
        plt.pause(0.1)

```

**代码运行截图**

使用jupyter notebook不能动态显示图

![](http://imgss.lovebingzi.com/tensorflow/%E6%9E%84%E5%BB%BA%E7%AE%80%E5%8D%95%E7%9A%84%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C.png)

**1、**在上述代码中x_data = np.linspace(-1,1,300)[:,np.newaxis]，关于numpy.linspace使用详解

![linspace](http://imgss.lovebingzi.com/tensorflow/linspace.png)

官网的例子

Examples

- np.linspace(2.0, 3.0, num=5) array([ 2. , 2.25, 2.5 , 2.75, 3. ])
- np.linspace(2.0, 3.0, num=5, endpoint=False) array([ 2. , 2.2, 2.4, 2.6, 2.8])
- np.linspace(2.0, 3.0, num=5, retstep=True) (array([ 2. , 2.25, 2.5 , 2.75, 3. ]), 0.25)

np.newaxis的功能是插入新维度，np.newaxis分别是在行或列上增加维度，原来是（6，）的数组，在行上增加维度变成（1,6）的二维数组，在列上增加维度变为(6,1)的二维数组

**2、**在tensorflow的使用中，经常会使用tf.reduce_mean平均数,tf.reduce_sum求和等函数，在函数中，有一个reduction_indices参数，表示函数的处理维度，直接上图，一目了然：

![reduction_indices](https://yunoss.chinadream.org/20180415/20180415_reduction_indices.png?OSSAccessKeyId=LTAIsG5eZOeImsiF&Expires=1523891041&Signature=KGfkmNKcP%2FjTtKe97D84V9jlVUs%3D)

需要注意的一点，在很多的时候，我们看到别人的代码中并没有reduction_indices这个参数，此时该参数取默认值None，将把input_tensor降到0维，也就是一个数。





