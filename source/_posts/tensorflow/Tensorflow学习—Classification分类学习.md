title: Tensorflow学习—Classification分类学习（手写数字例子）
author: 追梦人

toc: true

tags:

  - MachineLearning
  - tensorflow
  - classification
categories: []
date: 2018-04-17 15:25:00
---

# 使用神经网络识别手写数字

此教材来自[莫凡Python](https://morvanzhou.github.io/tutorials/machine-learning/tensorflow/)，本文旨在分享机器学习学习知识，如有侵权，可联系本人下架文章。

<!--more-->

人类视觉系统是世界上众多奇迹之一。看看下面的手写数字序列：

![digits](http://imgss.lovebingzi.com/mnist/digits.png)

大多数人毫不费力就能够认出这些数字为 504192。这么容易反而让人觉着迷惑了。在人类的每个脑半球中，有着一个初级视觉皮层，常称为 ，包含 1 亿 4 千万个神经元及数百亿条神经元间的连接。但是人类视觉不是就只有 ，还包括整个视觉皮层 —— 、、 和 —— 他们逐步地进行更加复杂的图像处理。人类的头脑就是一台超级计算机，通过数十亿年的进化不断地演变，最终能够极好地适应理解视觉世界的任务。识别手写数字也不是一件简单的事。尽管人类在理解我们眼睛展示出来的信息上非常擅长，但几乎所有的过程都是无意识地。所以，我们通常并不能体会自身视觉系统解决问题的困难。

如果你尝试写出计算机程序来识别诸如上面的数字，就会明显感受到视觉模式识别的困难。看起来人类一下子就能完成的任务变得特别困难。关于我们识别形状 —— “9 顶上有一个圈，右下方则是一条竖线”这样的简单直觉 —— 实际上算法上就很难轻易表达出来了。而在你试着让这些识别规则越发精准时，就会很快陷入各种混乱的异常或者特殊情形的困境中。看起来毫无希望。

神经网络以另一种方式看待这个问题。其主要思想是获取大量的手写数字，常称作训练样本，

![mnist_100_digits](http://imgss.lovebingzi.com/mnist/mnist_100_digits.png)

然后开发出一个可以从这些训练样本中进行学习的系统。换言之，神经网络使用样本来自动推断出识别手写数字的规则。另外，通过增加训练样本的数量，网络可以学到更多关于手写数字的知识，这样就能够提升自身的准确性。所以，上面例子中我们只是展出了 100 个训练数字样本，而通过使用数千或者数百万或者数十亿的训练样本我们也许能够得到更好的手写数字识别器。

本章我们将实现一个可以识别手写数字的神经网络。这个程序仅仅 74 行，不使用特别的神经网络库。

手写识别常常被当成学习神经网络的原型问题，因此我们聚焦在这个问题上。作为一个原型，它具备一个关键点：挑战性 —— 识别手写数字并不轻松 —— 但也不会难到需要超级复杂的解决方法，或者超大规模的计算资源。另外，这其实也是一种发展出诸如深度学习更加高级的技术的方法。所以，整本书我们都会持续地讨论手写数字识别问题。本书后面部分，我们会讨论这些想法如何用在其他计算机视觉的问题或者语音、自然语言处理和其他一些领域中。

当然，如果仅仅为了编写一个计算机程序来识别手写数字，本章的内容可以简短很多！但前进的道路上，我们将扩展出很多关于神经网络的关键的思想，其中包括两个重要的人工神经元（感知器和 S 型神经元），以及标准的神经网络学习算法，即随机梯度下降算法。自始至终，我专注于解释事情的原委，并构筑你对神经网络的直观感受。这需要一个漫长的讨论，而不是仅仅介绍些基本的技巧，但这对于更深入的理解是值得的。作为收益，在本章的最后，我们会准备好了解什么是深度学习，以及它为什么很重要。

## 1 一个简单的分类手写数字的网络

定义了神经网络后，让我们回到手写识别。我们能将识别手写数字分成两个子问题。首先，我们想办法将一个包含很多数字的图像分成一系列独立的图像，每张包含唯一的数字。比如我们将把下面图像

![digits](http://imgss.lovebingzi.com/mnist/digits.png)

分成6个分离的小图像，

![digits_separate](http://imgss.lovebingzi.com/mnist/digits_separate.png)

我们人类能够很容易解决这个*分段问题*，但是对于计算机程序如何正确的分离图像却是一个挑战。然后，一旦这幅图像被分离，程序需要将各个数字进行分类。因此，比如，我们希望程序能够将上面的第一个数字，

![](http://imgss.lovebingzi.com/mnist/mnist_first_digit.png)

识别为5。

我们主要关注在第二个问题，也就是，分类这些独立的数字图像。我们这么做是因为一旦你能够找到一个好方式来分类独立的图像，分离问题就不是那么难解决了。有许多途径能够解决分离问题，一种途径是试验很多不同的分离图像方法，并采用独立图像分类器给每个分离试验打分。如果独立数字分类器坚信某种分离方式，那么打分较高。如果分类器在某些片断上问题很多，那么得分就低。这个思想就是如果分类器分类效果很有问题，那么很可能是分离方式不对造成。这种想法和一些变型能够很好解决分离问题。因此无须担心分离，我们将集中精力开发一个神经网络，它能解决更有趣和复杂的问题，即识别独立的手写数字。

为了识别这些数字，我们将采用三层神经网络：

![](http://imgss.lovebingzi.com/mnist/tikz12.png)

网络的输入层含有输入像素编码的神经元。跟下节讨论的一样，我们用于网络训练的数据将包含许多28 by 28像素手写数字扫描图像，因此输入层包含
$$
784=28×28
$$
个神经元。为了简化，我们在上图中忽略了许多输入神经元。输入像素是灰度值，白色值是0.0，黑色值是1.0，它们之间的值表明灰度逐渐变暗的程度。

网络的第二层是隐含层。我们将其神经元的数量表示为nn，后面还对其采用不同的nn值进行实验。这个例子中使用了一个小的隐含层，只包含n=15个神经元。

网络的输出层包含10个神经元。如果第一个神经元被触发，例如它有一个≈1的输出，那么这就表明这个网络认为识别的数字是0。更精确一点，我们将神经元输出标记为0到9，然后找出具有最大激励值的神经元。如果这个神经元是6，那么这个网络将认为输入数字是6。对于其他神经元也有类似结果。

你可能想知道为什么用10个输出神经元。别忘了，网络的目标是识别出输入图像对应的数字（0,1,2,…,9）。一种看起来更自然的方法是只用44个输出神经元，每个神经元都当做一个二进制值，取值方式取决于神经元输出接近0，还是1。4个神经元已经足够用来编码输出，因为2的四次方=16

大于输入数字的10种可能值。（为什么我们的网络使用10个神经元呢？是否这样效率太低？最终的理由是以观察和实验为依据的：我们能尝试两种网络设计，最后结果表明对于这个特殊问题，具有10个输出神经元的网络能比只有4个输出神经元的网络更好的学习识别手写数字。这使得我们想知道*为什么*具有10个输出神经元的网络效果更好。这里是否有一些启发，事先告诉我们应该用10个输出神经元，而不是4个输出神经元？

为了弄懂我们为什么这么做，从原理上想清楚神经网络的工作方式很重要。首先考虑我们使用10个输出神经元的情况。让我们集中在第一个输出神经元上，它能试图确定这个手写数字是否为0。它通过对隐含层的凭据进行加权和求出。隐含层的神经元做了什么呢？假设隐含层中第一个神经元目标是检测到是否存在像下面一样的图像：

![](http://imgss.lovebingzi.com/mnist/mnist_top_left_feature.png)

它能够对输入图像与上面图像中像素重叠部分进行重加权，其他像素轻加权来实现。相似的方式，对隐含层的第二、三和四个神经元同样能检测到如下所示的图像：

![](http://imgss.lovebingzi.com/mnist/mnist_other_features.png)

你可能已经猜到，这四幅图像一起构成了我们之前看到的0的图像：因此如果这四个神经元一起被触发，那么我们能够推断出手写数字是0。当然，这些不*只*是我们能推断出0的凭据类别——我们也可以合理的用其他方式得到0（通过对上述图像的平移或轻微扭曲）。但是至少这种方式推断出输入是0看起来是安全的。

假设神经网络按这种方式工作，我们能对为什么10个输出神经元更好，而不是4个，给出合理的解释。如果我们有4个输出神经元，那么第一个输出神经元将试着确定什么是数字图像中最重要的位。这里没有很容易的方式来得到如上图所示简单的形状的重要位。很难想象，这里存在任何一个很好的根据来说明数字的形状组件与输出重要位的相关性。

现在，就像所说的那样，这就是一个启发式。没有什么能说明三层神经网络会按照我描述的那样运作，这些隐含层能够确定简单的形状组件。可能一个更聪明的学习算法将找到权重赋值以便我们使用44个输出神经元。但是我描述的这个启发式方法已经运作很好，能够节约很多时间来设计更好的神经元网络架构。

## 2 梯度下降学习法

现在我们已经有一个设计好的神经网络，它怎样能够学习识别数字呢？首要的事情是我们需要一个用于学习的数据集——也叫做训练数据集。我们将使用[MNIST数据集](http://yann.lecun.com/exdb/mnist/)，它包含成千上万的手写数字图像，同时还有它们对应的数字分类。MNIST的名字来源于[NIST](http://en.wikipedia.org/wiki/National_Institute_of_Standards_and_Technology)（美国国家标准与技术研究所）收集的两个修改后的数据集。这里是一些来自于MNIST的图像：

![](http://imgss.lovebingzi.com/mnist/digits.png)

就像你看到的，事实上这些数字和[本章开始](http://tensorfly.cn/special/deeplearning/chap1.html#complete_zero)展示的用于识别的图像一样。当然，测试我们的网络时候，我们会让它识别不在训练集中的图像！

MNIST数据来自两部分。第一部分包含60,000幅用于训练的图像。这些图像是扫描250位人员的手写样本得到的，他们一半是美国人口普查局雇员，一半是高中学生。这些图像都是28乘28像素的灰度图。MNIST数据的第二部分包含10,000幅用于测试的图像。它们也是28乘28的灰度图像。我们将使用这些测试数据来评估我们用于学习识别数字的神经网络。为了得到很好的测试性能，测试数据来自和训练数据*不同*的250位人员（仍然是来自人口普查局雇员和高中生）。 这帮助我们相信这个神经网络能够识别在训练期间没有看到的其他人写的数字。

**关于梯度学习法可以参考[这里](https://www.dreamer.im/2018/03/20/ML&DL/MachineLearning%E2%80%942%E3%80%81%E7%BA%BF%E6%80%A7%E5%9B%9E%E5%BD%92/)**

## 3 使用神经网络分类手写数字

好了，让我们使用随机梯度下降算法和MNIST训练数据来写一个能够学习识别手写数字的程序。首先我们需要获得MNIST数据。如果你是`git`的使用者，你可以通过克隆这本书的源代码库来获得这些数据，

```
git clone https://github.com/mnielsen/neural-networks-and-deep-learning.git
```

如果你不使用git，你可以通过[这里](https://github.com/mnielsen/neural-networks-and-deep-learning/archive/master.zip)下载所需的数据和源代码。

顺便说一下，早些时候我在描述MNIST数据时，我说过它被分成了两份，其中60000幅图片用于训练，另外10000幅用于测试。那是MNIST官方的陈述。

```python
import tensorflow as tf
from tensorflow.examples.tutorials.mnist import input_data

mnist = input_data.read_data_sets('MNIST_data',one_hot=True)

#定义test数据集与train训练集的数据准确率
def compute_accuracy(v_xs,v_ys):
    global prediction
    y_pre = sess.run(prediction,feed_dict={xs:v_xs})
    correct_prediction = tf.equal(tf.argmax(y_pre,1),tf.argmax(v_ys,1))
    #tf.cast：用于改变某个张量的数据类型
    accuracy = tf.reduce_mean(tf.cast(correct_prediction,tf.float32)) 
    result = sess.run(accuracy,feed_dict={xs:v_xs,ys:v_ys})
    return result

def add_layer(inputs,in_size,out_size,activation_function=None): #输入参数，行列矩阵维数，激励函数
     #定义权重为行(in_zise)列(out_size)的随机矩阵
    Weights = tf.Variable(tf.random_normal([in_size,out_size])) 
    biases = tf.Variable(tf.zeros([1,out_size]) + 0.1)           #定于误差
    Wx_plus_b = tf.matmul(inputs,Weights) + biases               #线性运算，W*x+biases
    if activation_function is None:
        outputs = Wx_plus_b
    else:
        outputs = activation_function(Wx_plus_b)
    return outputs
    
#define placeholder for inputs to network
xs = tf.placeholder(tf.float32,[None,784])  #28*28
ys = tf.placeholder(tf.float32,[None,10])

#add output layer
#一般在分类学习用，使用tf.nn.softmax
prediction = add_layer(xs,784,10,activation_function=tf.nn.softmax)

#the error between prediction and real data
#loss函数（即最优化目标函数）选用交叉熵函数。交叉熵用来衡量预测值和真实值的相似程度，如果完全相同，它们的交叉熵等于零。
cross_entropy = tf.reduce_mean(-tf.reduce_sum(ys * tf.log(prediction),reduction_indices=[1]))  #loss
train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)

sess = tf.Session()
#important step
#sess.run(tf.initialize_all_variables())
sess.run(tf.global_variables_initializer())

for i in range(1000):
    batch_xs,batch_ys = mnist.train.next_batch(100)
    sess.run(train_step,feed_dict={xs:batch_xs,ys:batch_ys})
    if i % 50 ==0:
        print(compute_accuracy(mnist.test.images,mnist.test.labels))
```

运行截图：

![](http://imgss.lovebingzi.com/mnist/1.png)

![](http://imgss.lovebingzi.com/mnist/output.gif)

可以看到，test集的数据准确率通过学习，准确率越来越高。