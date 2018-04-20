  title: Tensorflow学习—利用CNN来优化上节神经网络

  author: 追梦人

  toc: true

  categories: []

  date: 2018-4-20 11:36:00

  tags:

  - MachineLearning
  - tensorflow
  - ConvolutionNetwork

---

# 利用卷积神经网络来优化上节分类学习的准确度

此教材来自[莫凡Python](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/)，本文旨在分享机器学习学习知识，如有侵权，可联系本人下架文章。

学习资料:

- 为 TF 2017 打造的[新版可视化教学代码](https://github.com/MorvanZhou/Tensorflow-Tutorial)
- 机器学习-简介系列什么是[CNN](https://www.dreamer.im/2018/03/13/ML&DL/DeepLearning-for-Convolutional-Neural-Networks/)

<!--more-->

## 1、定义卷积层的 weight bias 

首先我们导入

```python
import tensorflow as tf
```

采用的数据集依然是`tensorflow`里面的`mnist`数据集

我们需要先导入它

```python
python from tensorflow.examples.tutorials.mnist import input_data
```

本次课程代码用到的数据集就是来自于它

```python
mnist=input_data.read_data_sets('MNIST_data',one_hot=true)
```

接着呢，我们定义`Weight`变量，输入`shape`，返回变量的参数。其中我们使用`tf.truncted_normal`产生随机变量来进行初始化:

```python
def weight_variable(shape): 
	inital=tf.truncted_normal(shape,stddev=0.1)
	return tf.Variable(initial)
```

tf.truncated_normal(shape, mean, stddev) :shape表示生成张量的维度，mean是均值，stddev是标准差。这个函数产生正太分布，均值和标准差自己设定。这是一个截断的产生正太分布的函数，就是说产生正太分布的值如果与均值的差值大于两倍的标准差，那就重新生成。和一般的正太分布的产生随机数据比起来，这个函数产生的随机数与均值的差距不会超过两倍的标准差，但是一般的别的函数是可能的。

同样的定义`biase`变量，输入`shape` ,返回变量的一些参数。其中我们使用`tf.constant`常量函数来进行初始化:

```python
def bias_variable(shape): 
	initial=tf.constant(0.1,shape=shape) 
	return tf.Variable(initial)
```

定义卷积，`tf.nn.conv2d`函数是`tensoflow`里面的二维的卷积函数，`x`是图片的所有参数，`W`是此卷积层的权重，然后定义步长`strides=[1,1,1,1]`值，`strides[0]`和`strides[3]`的两个1是默认值，中间两个1代表padding时在x方向运动一步，y方向运动一步，padding采用的方式是`SAME`。

```python
def conv2d(x,W):
	return tf.nn.conv2d(x,W,strides=[1,1,1,1]，padding='SAME') 
```

## 2、定义pooling

接着定义池化`pooling`，为了得到更多的图片信息，padding时我们选的是一次一步，也就是`strides[1]=strides[2]=1`，这样得到的图片尺寸没有变化，而我们希望压缩一下图片也就是参数能少一些从而减小系统的复杂度，因此我们采用`pooling`来稀疏化参数，也就是卷积神经网络中所谓的下采样层。`pooling `有两种，一种是最大值池化，一种是平均值池化，本例采用的是最大值池化`tf.max_pool()`。池化的核函数大小为2x2，因此`ksize=[1,2,2,1]`，步长为2，因此`strides=[1,2,2,1]`:

```python
def max_poo_2x2(x): 
	return tf.nn.max_pool(x,ksize=[1,2,2,1],strides=[1,2,2,1])
```



这一次我们一层层的加上了不同的 layer. 分别是:

1. convolutional layer1 + max pooling;
2. convolutional layer2 + max pooling;
3. fully connected layer1 + dropout;
4. fully connected layer2 to prediction.

## 3、图片处理

首先呢，我们定义一下输入的`placeholder`

```python
xs=tf.placeholder(tf.float32,[None,784])
ys=tf.placeholder(tf.float32,[None,10])
```

我们还定义了`dropout`的`placeholder`，它是解决过拟合的有效手段

```python
keep_prob=tf.placeholder(tf.float32)
```

接着呢，我们需要处理我们的`xs`，把`xs`的形状变成`[-1,28,28,1]`，-1代表先不考虑输入的图片例子多少这个维度，后面的1是channel的数量，因为我们输入的图片是黑白的，因此channel是1，例如如果是RGB图像，那么channel就是3。

```python
x_image=tf.reshape(xs,[-1,28,28,1])
```

## 4、建立卷积层

接着我们定义第一层卷积,先定义本层的`Weight`,本层我们的卷积核patch的大小是5x5，因为黑白图片channel是1所以输入是1，输出是32个featuremap

```python
W_conv1=weight_variable([5,5,1,32])
```

接着定义`bias`，它的大小是32个长度，因此我们传入它的`shape`为`[32]`

```python
b_conv1=bias_variable([32])
```

定义好了`Weight`和`bias`，我们就可以定义卷积神经网络的第一个卷积层`h_conv1=conv2d(x_image,W_conv1)+b_conv1`,同时我们对`h_conv1`进行非线性处理，也就是激活函数来处理喽，这里我们用的是`tf.nn.relu`（修正线性单元）来处理，要注意的是，因为采用了`SAME`的padding方式，输出图片的大小没有变化依然是28x28，只是厚度变厚了，因此现在的输出大小就变成了28x28x32

```python
h_conv1=tf.nn.relu(conv2d(x_image,W_conv1)+b_conv1)
```

最后我们再进行`pooling`的处理就ok啦，经过`pooling`的处理，输出大小就变为了14x14x32

```python
h_pool=max_pool_2x2(h_conv1)
```

接着呢，同样的形式我们定义第二层卷积，本层我们的输入就是上一层的输出，本层我们的卷积核patch的大小是5x5，有32个featuremap所以输入就是32，输出呢我们定为64

```python
W_conv2=weight_variable([5,5,32,64])
b_conv2=bias_variable([64])
```

接着我们就可以定义卷积神经网络的第二个卷积层，这时的输出的大小就是14x14x64

```python
h_conv2=tf.nn.relu(conv2d(h_pool1,W_conv2)+b_conv2)
```

最后也是一个pooling处理，输出大小为7x7x64

```python
h_pool2=max_pool_2x2(h_conv2)
```

## 5、建立全连接层

好的，接下来我们定义我们的 fully connected layer,

进入全连接层时, 我们通过`tf.reshape()`将`h_pool2`的输出值从一个三维的变为一维的数据, -1表示先不考虑输入图片例子维度, 将上一个输出结果展平.

```python
#[n_samples,7,7,64]->>[n_samples,7*7*64]
h_pool2_flat=tf.reshape(h_pool2,[-1,7*7*64]) 
```

此时`weight_variable`的`shape`输入就是第二个卷积层展平了的输出大小: 7x7x64， 后面的输出size我们继续扩大，定为1024

```python
W_fc1=weight_variable([7*7*64,1024]) 
b_fc1=bias_variable([1024])
```

然后将展平后的`h_pool2_flat`与本层的`W_fc1`相乘（注意这个时候不是卷积了）

```python
h_fc1=tf.nn.relu(tf.matmul(h_pool2_flat,W_fc1)+b_fc1)
```

如果我们考虑过拟合问题，可以加一个dropout的处理

```python
h_fc1_drop=tf.nn.dropout(h_fc1,keep_drop)
```

接下来我们就可以进行最后一层的构建了，好激动啊, 输入是1024，最后的输出是10个 (因为mnist数据集就是[0-9]十个类)，prediction就是我们最后的预测值

```python
W_fc2=weight_variable([1024,10]) b_fc2=bias_variable([10])
```

然后呢我们用softmax分类器（多分类，输出是各个类的概率）,对我们的输出进行分类

```python
prediction=tf.nn.softmax(tf.matmul(h_fc1_dropt,W_fc2),b_fc2)
```

## 6、选优化方法

接着呢我们利用交叉熵损失函数来定义我们的cost function

```python
cross_entropy=tf.reduce_mean(
    -tf.reduce_sum(ys*tf.log(prediction),
    reduction_indices=[1]))
```

我们用`tf.train.AdamOptimizer()`作为我们的优化器进行优化，使我们的`cross_entropy`最小

```python
train_step=tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
```

接着呢就是和之前视频讲的一样喽 定义`Session`,初始化变量,好啦接着就是训练数据啦，我们假定训练`1000`步，每`50`步输出一下准确率， 注意`sess.run()`时记得要用`feed_dict`给我们的众多 `placeholder` 喂数据哦.

## 7、完整代码

```python
import tensorflow as tf
from tensorflow.examples.tutorials.mnist import input_data

mnist = input_data.read_data_sets('MNIST_data', one_hot=True)


def compute_accuracy(v_xs, v_ys):
    global prediction
    y_pre = sess.run(prediction, feed_dict={xs: v_xs, keep_prob: 1})
    correct_prediction = tf.equal(tf.argmax(y_pre, 1), tf.argmax(v_ys, 1))
    accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))  # tf.cast：用于改变某个张量的数据类型
    result = sess.run(accuracy, feed_dict={xs: v_xs, ys: v_ys, keep_prob: 1})
    return result


def weight_variable(shape):
    initial = tf.truncated_normal(shape, stddev=0.1)
    return tf.Variable(initial)


def bias_variable(shape):
    initial = tf.constant(0.1, shape=shape)
    return tf.Variable(initial)


def conv2d(x, W):
    # stride[1,x_movement,y_movement,1]
    return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')


def max_pool_2x2(x):
    return tf.nn.max_pool(x, ksize=[1, 2, 2, 1], strides=[1, 2, 2, 1], padding='SAME')


# define placeholder for inputs to network
xs = tf.placeholder(tf.float32, [None, 784])  # 28*28
ys = tf.placeholder(tf.float32, [None, 10])
keep_prob = tf.placeholder(tf.float32)
x_image = tf.reshape(xs, [-1, 28, 28, 1])

# conv1 layer
W_conv1 = weight_variable([5, 5, 1, 32])  # patch 5x5,in size 1,out size 32
b_conv1 = bias_variable([32])
h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)  # output size 28x28x32
h_pool1 = max_pool_2x2(h_conv1)  # output size 14x14x32

# conv2_layer
W_conv2 = weight_variable([5, 5, 32, 64])  # patch 5x5,in size 32,out size 64
b_conv2 = bias_variable([64])
h_conv2 = tf.nn.relu(conv2d(h_pool1, W_conv2) + b_conv2)  # output size 14x14x64
h_pool2 = max_pool_2x2(h_conv2)  # output size 7x7x64

# func1_layer
W_fc1 = weight_variable([7 * 7 * 64, 1024])
b_fc1 = bias_variable([1024])
h_pool2_flat = tf.reshape(h_pool2, [-1, 7 * 7 * 64])  # [n_samples,7,7,64] -> [n_samples,7*7*64]
h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat, W_fc1) + b_fc1)
h_fc1_drop = tf.nn.dropout(h_fc1, keep_prob)

# func2_layer
W_fc2 = weight_variable([1024, 10])
b_fc2 = bias_variable([10])
prediction = tf.nn.softmax(tf.matmul(h_fc1_drop, W_fc2) + b_fc2)

# the error between prediction and real data
cross_entropy = tf.reduce_mean(-tf.reduce_sum(ys * tf.log(prediction), reduction_indices=[1]))  # loss
train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)

sess = tf.Session()
# important step
sess.run(tf.initialize_all_variables())

for i in range(1000):
    batch_xs, batch_ys = mnist.train.next_batch(100)
    sess.run(train_step, feed_dict={xs: batch_xs, ys: batch_ys, keep_prob: 0.5})
    if i % 50 == 0:
        print(compute_accuracy(mnist.test.images, mnist.test.labels))
```

运行结果：

经过漫长的等待，我的CPU爆载后（无辜脸）

![](http://imgss.lovebingzi.com/cnn/output.png)

![](http://imgss.lovebingzi.com/cnn/output.gif)

