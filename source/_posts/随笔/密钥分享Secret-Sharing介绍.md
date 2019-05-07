title: 密钥分享Secret Sharing介绍
author: 追梦人

toc: true

tags:

  - SecretSharing
  - 密码学
categories: []
date: 2019-05-07 16:39:00

---

# 密钥分享Secret Sharing介绍

本文转载自**李天天**知乎专栏安全计算和密码学，原文链接：[点击这儿](https://zhuanlan.zhihu.com/p/44999983)！

参与双方通过传输加密电路实现安全计算。理论上各种计算都可以用这种方法实现。对于各种纯粹由位运算（就是AND、OR、XOR这些）组成的算法（如比较操作或AES加密），GC效率是比较高的。但有一个问题是，即便一些常见的算术操作（如乘法、乘方等），电路也非常复杂，这意味着很多常见算法GC应付起来都很吃力。比如下面是两位整数的乘法电路，我们平时用的都是32位甚至64位乘法，还包括浮点运算等，直接用GC解决，效率是不敢恭维的。而现实生活中很多常用的算法，如目前比较火的机器学习深度学习算法包含了大量的浮点数/定点数乘法、除法、指数运算等等，纯靠GC是不能指望的。而本文介绍的密钥分享（secret sharing）则正好对算术操作比较拿手。

<!-- more -->

## 密钥分享原理

密钥分享的基本思路是将每个数字 ![x](https://www.zhihu.com/equation?tex=x) 拆散成多个数 ![x_1,x_2,\dots,x_n](https://www.zhihu.com/equation?tex=x_1%2Cx_2%2C%5Cdots%2Cx_n) ，并将这些数分发到多个参与方 ![S_1,S_2,\dots,S_n](https://www.zhihu.com/equation?tex=S_1%2CS_2%2C%5Cdots%2CS_n) 那里。然后每个参与方拿到的都是原始数据的一部分，一个或少数几个参与方无法还原出原始数据，只有大家把各自的数据凑在一起时才能还原真实数据。计算时，各参与方直接用它自己本地的数据进行计算，并且在适当的时候交换一些数据（交换的数据本身看起来也是随机的，不包含关于原始数据的信息），计算结束后的结果仍以secret sharing的方式分散在各参与方那里，并在最终需要得到结果的时候将某些数据合起来。这样的话，密钥分享便保证了计算过程中各个参与方看到的都是一些随机数，但最后仍然算出了想要的结果。

![EsJiyd.jpg](https://s2.ax1x.com/2019/05/07/EsJiyd.jpg)

## 密钥分享运作方式

那密钥分享具体是怎么运作的呢？我们先从一个最简单的方法讲起。假设A这个人有一个秘密数字 ![x](https://www.zhihu.com/equation?tex=x) ，他想将其分发到![S_1,S_2,\dots,S_n](https://www.zhihu.com/equation?tex=S_1%2CS_2%2C%5Cdots%2CS_n)那里。那么A首先要做的便是生成 ![n-1](https://www.zhihu.com/equation?tex=n-1) 个随机数 ![r_1, r_2, \dots, r_n](https://www.zhihu.com/equation?tex=r_1%2C+r_2%2C+%5Cdots%2C+r_n) ，然后计算第 ![n](https://www.zhihu.com/equation?tex=n) 个数 ![r_n = x - \sum_{i=1}^{n-1}r_i](https://www.zhihu.com/equation?tex=r_n+%3D+x+-+%5Csum_%7Bi%3D1%7D%5E%7Bn-1%7Dr_i) ，最后A令 ![x_1=r_1, x_2 = r_2, \dots, x_n = r_n](https://www.zhihu.com/equation?tex=x_1%3Dr_1%2C+x_2+%3D+r_2%2C+%5Cdots%2C+x_n+%3D+r_n) ，并将它们发给![S_1,S_2,\dots,S_n](https://www.zhihu.com/equation?tex=S_1%2CS_2%2C%5Cdots%2CS_n)。上面这种简单的方法具有如下几条性质：

1. 各个数字 ![x_1, x_2, \dots, x_n](https://www.zhihu.com/equation?tex=x_1%2C+x_2%2C+%5Cdots%2C+x_n) 都是随机分布的，单独一个或若干个并不泄露任何信息；
2. 当所有![x_1, x_2, \dots, x_n](https://www.zhihu.com/equation?tex=x_1%2C+x_2%2C+%5Cdots%2C+x_n)合在一起时，可以还原 ![x](https://www.zhihu.com/equation?tex=x) ，因为 ![x = \sum_{i=1}^n x_i](https://www.zhihu.com/equation?tex=x+%3D+%5Csum_%7Bi%3D1%7D%5En+x_i) ;
3. 这种方案具有加法同态的性质，也就是说，各参与方可以在不交换任何数据的情况下直接计算对秘密数据求和。什么意思呢？假设还有另一个人B，他也有一个秘密数字 ![y](https://www.zhihu.com/equation?tex=y) ,并且和A一起将数据分发给了![S_1,S_2,\dots,S_n](https://www.zhihu.com/equation?tex=S_1%2CS_2%2C%5Cdots%2CS_n)

![EsJAeI.jpg](https://s2.ax1x.com/2019/05/07/EsJAeI.jpg)

## 密钥分享加法运算

为了做加法， ![S_1](https://www.zhihu.com/equation?tex=S_1) 计算 ![z_1=x_1+y_1](https://www.zhihu.com/equation?tex=z_1%3Dx_1%2By_1) ,![S_2](https://www.zhihu.com/equation?tex=S_2) 计算 ![z_2=x_2+y_2 ](https://www.zhihu.com/equation?tex=z_2%3Dx_2%2By_2+) , …, ![S_n](https://www.zhihu.com/equation?tex=S_n) 计算 ![z_n=x_n+y_n](https://www.zhihu.com/equation?tex=z_n%3Dx_n%2By_n) ,

每个参与方都只对本地的随机数进行操作，不交换数据。而且根据secret sharing的性质，我们其实可以看到： ![\sum_{i=1}^nz_i = \sum_{i=1}^nx_i + \sum_{i=1}^ny_i](https://www.zhihu.com/equation?tex=%5Csum_%7Bi%3D1%7D%5Enz_i+%3D+%5Csum_%7Bi%3D1%7D%5Enx_i+%2B+%5Csum_%7Bi%3D1%7D%5Eny_i) 。也就是说，我们得到的是 ![z= x+y](https://www.zhihu.com/equation?tex=z%3D+x%2By) 的密钥分享，而这个求和的结果可以不暴露出来，继续用来做其他事情。

### 阈值密钥分享

上面是一个简单的密钥分享的方法，满足了加法同态，且保证了只有n个参与方全部联合才能把数据解开。但有时候我们并不希望必须凑齐n个人才能解开，一方面是因为数据是分散在多个人手里的，要是有一个人不小心掉线了甚至是故意使坏，那数据就无法恢复了，另一方面很多时候我们不需要这么强的安全性，比如我们可以相信10个人里面至少一半是好人，而好人是不会偷偷把数据解开的，那么我们只需要保证4个或更少的人无法将数据解开就行了，而只有凑齐了5个或更多的人才能将数据解开。这种密钥分享叫做阈值密钥分享（threshold secret sharing）。

更具体地说，我们可以定义一种名为 ![(t,n)](https://www.zhihu.com/equation?tex=%28t%2Cn%29) 阈值密钥分享的方案，此类方案允许任意 ![t](https://www.zhihu.com/equation?tex=t)

个参与方将秘密数据解开，但任何不多于 ![t-1](https://www.zhihu.com/equation?tex=t-1) 个参与方的小团体都无法将秘密数据解开。前面提到的那种简单方案其实是 ![t=n](https://www.zhihu.com/equation?tex=t%3Dn) 时的特殊情况。Shamir大神在1979年就提出了阈值密钥分享方案，且该方案支持任意的 ![t](https://www.zhihu.com/equation?tex=t) 。该方案运作方式如下：假设A想要使用 ![(t,n)](https://www.zhihu.com/equation?tex=%28t%2Cn%29) 阈值密钥分享技术将某秘密数字 ![s](https://www.zhihu.com/equation?tex=s) 分享给![S_1,S_2,\dots,S_n](https://www.zhihu.com/equation?tex=S_1%2CS_2%2C%5Cdots%2CS_n)，那么他首先生成一个 ![t-1](https://www.zhihu.com/equation?tex=t-1) 次多项式多项式 ![f(x)=a_0 + a_1 x + a_2 x^2 + \dots + a_{t-1}x^{t-1}](https://www.zhihu.com/equation?tex=f%28x%29%3Da_0+%2B+a_1+x+%2B+a_2+x%5E2+%2B+%5Cdots+%2B+a_%7Bt-1%7Dx%5E%7Bt-1%7D) ，其中 ![a_0](https://www.zhihu.com/equation?tex=a_0) 就等于要分享的秘密数字 ![s](https://www.zhihu.com/equation?tex=s) ，而 ![a_1, a_2, \dots, a_{t-1}](https://www.zhihu.com/equation?tex=a_1%2C+a_2%2C+%5Cdots%2C+a_%7Bt-1%7D) ，则是A生成的随机数。随后A只需将![s_1 = f(1), s_2 = f(2), \dots, s_n = f(n)](https://www.zhihu.com/equation?tex=s_1+%3D+f%281%29%2C+s_2+%3D+f%282%29%2C+%5Cdots%2C+s_n+%3D+f%28n%29) 分别发给![S_1,S_2,\dots,S_n](https://www.zhihu.com/equation?tex=S_1%2CS_2%2C%5Cdots%2CS_n)即可。到了这一步，稍微有点线性代数基础的同学应该很容易看出来， ![f(1), f(2), \dots, f(n)](https://www.zhihu.com/equation?tex=f%281%29%2C+f%282%29%2C+%5Cdots%2C+f%28n%29) 中任意 ![t](https://www.zhihu.com/equation?tex=t) 个凑在一起都可以解出，而任意 ![t-1](https://www.zhihu.com/equation?tex=t-1) 个凑在一起都无法得到 ![a_0](https://www.zhihu.com/equation?tex=a_0) （即 ![s](https://www.zhihu.com/equation?tex=s) ）的确切解。通过这一点便达到了 ![(t,n)](https://www.zhihu.com/equation?tex=%28t%2Cn%29) 阈值的要求。Shamir密钥分享方法也是满足加法同态的（因为多项式本身满足这一性质）。

## 密钥分享乘法运算

说到这里，大家可以看到我们可以很容易地使用密钥分享技术在参与方不交换任何信息的情况下完成保护隐私的加法操作，但本文一开始提到的更重要的乘法操作呢？在完全不交换信息的情况下，要完成乘法是很难实现的，但如果在计算前或计算过程中适当交换信息，要完成乘法操作却有不少解决方案。

我们先考虑最简单的一种情况：一个秘密数字 ![x](https://www.zhihu.com/equation?tex=x) 和一个公开数字 ![y](https://www.zhihu.com/equation?tex=y) 相乘，目标是得到一个数字 ![z](https://www.zhihu.com/equation?tex=z) 的密钥分享，其中满足 ![z=x \times y](https://www.zhihu.com/equation?tex=z%3Dx+%5Ctimes+y) 。这个做起来其实挺简单的。假设我们使用最开始说的那种简单的密钥分享方法，即，那么我们的目标就是让![S_1,S_2,\dots,S_n](https://www.zhihu.com/equation?tex=S_1%2CS_2%2C%5Cdots%2CS_n)分别得到 ![z_1, z_2, \dots, z_n](https://www.zhihu.com/equation?tex=z_1%2C+z_2%2C+%5Cdots%2C+z_n)，且满足 ![z = \sum_{i=1}^nz_i](https://www.zhihu.com/equation?tex=z+%3D+%5Csum_%7Bi%3D1%7D%5Enz_i) 。要达成此目标，只需让 ![S_1](https://www.zhihu.com/equation?tex=S_1) 计算 ![z_1 = x_1 \times y](https://www.zhihu.com/equation?tex=z_1+%3D+x_1+%5Ctimes+y) ,![S_2](https://www.zhihu.com/equation?tex=S_2) 计算 ![z_2 = x_2 \times y](https://www.zhihu.com/equation?tex=z_2+%3D+x_2+%5Ctimes+y) ,..., ![S_n](https://www.zhihu.com/equation?tex=S_n) 计算 ![z_n = x_n \times y](https://www.zhihu.com/equation?tex=z_n+%3D+x_n+%5Ctimes+y) ,这个应该很容易理解。好吧这里仍然不需要参与方交换信息。

但如果 ![y](https://www.zhihu.com/equation?tex=y) 不是公开数字呢？也就是说![S_1,S_2,\dots,S_n](https://www.zhihu.com/equation?tex=S_1%2CS_2%2C%5Cdots%2CS_n)只拥有 ![y_1, y_2, \dots, y_n](https://www.zhihu.com/equation?tex=y_1%2C+y_2%2C+%5Cdots%2C+y_n) ，而不知道 ![y](https://www.zhihu.com/equation?tex=y)的确切值。这时候上面说的方法就不管用了。在不交换信息的情况下，![S_1,S_2,\dots,S_n](https://www.zhihu.com/equation?tex=S_1%2CS_2%2C%5Cdots%2CS_n)只能分别算出 ![x_1 \times y_1, x_2 \times y_2, \dots, x_n \times y_n](https://www.zhihu.com/equation?tex=x_1+%5Ctimes+y_1%2C+x_2+%5Ctimes+y_2%2C+%5Cdots%2C+x_n+%5Ctimes+y_n) ，但无法计算交叉项。欲求交叉项，必有信息交换，不过这个信息的交换，既可以发生在计算前，也可以发生在计算过程中，或者两个阶段都有信息交换。下面介绍一下如何使用预计算生成乘积元组的方法解决乘法问题。

我们假设有某种神奇的方法（具体怎么做就不展开了），使得![S_1,S_2,\dots,S_n](https://www.zhihu.com/equation?tex=S_1%2CS_2%2C%5Cdots%2CS_n)能在计算发生前预先得到两个随机数 ![a](https://www.zhihu.com/equation?tex=a) 和 ![b](https://www.zhihu.com/equation?tex=b) 的秘密分享，以及 ![a](https://www.zhihu.com/equation?tex=a) 和 ![b](https://www.zhihu.com/equation?tex=b) 的乘积 ![c](https://www.zhihu.com/equation?tex=c) 的秘密分享，而且它们都不知道 ![a](https://www.zhihu.com/equation?tex=a) 和 ![b](https://www.zhihu.com/equation?tex=b) 和 ![c](https://www.zhihu.com/equation?tex=c) 的真实值，如下图所示：

![EsJPQH.jpg](https://s2.ax1x.com/2019/05/07/EsJPQH.jpg)

现在有A和B分别分享了两个数字 ![x](https://www.zhihu.com/equation?tex=x) 和 ![y](https://www.zhihu.com/equation?tex=y) ，参与方需要算出 ![x](https://www.zhihu.com/equation?tex=x) 和 ![y](https://www.zhihu.com/equation?tex=y) 的乘积 ![z](https://www.zhihu.com/equation?tex=z) 的密钥分享。这时候可以借助前面生成的随机乘积元组。我们先令 ![s=x-a](https://www.zhihu.com/equation?tex=s%3Dx-a) 以及 ![t = y - b](https://www.zhihu.com/equation?tex=t+%3D+y+-+b) ，然后我们可以看到

![x \times y = (x - a + a) \times (y - b + b) = (s+a) \times (t + b) = s \times t + s \times b + t \times a + c](https://www.zhihu.com/equation?tex=x+%5Ctimes+y+%3D+%28x+-+a+%2B+a%29+%5Ctimes+%28y+-+b+%2B+b%29+%3D+%28s%2Ba%29+%5Ctimes+%28t+%2B+b%29+%3D+s+%5Ctimes+t+%2B+s+%5Ctimes+b+%2B+t+%5Ctimes+a+%2B+c)

参与方![S_1,S_2,\dots,S_n](https://www.zhihu.com/equation?tex=S_1%2CS_2%2C%5Cdots%2CS_n)可以联合起来将 ![s](https://www.zhihu.com/equation?tex=s) 和 ![t](https://www.zhihu.com/equation?tex=t) 的值解开，由于 ![a](https://www.zhihu.com/equation?tex=a) 和 ![b](https://www.zhihu.com/equation?tex=b) 都是值未知的随机数，因此 ![s](https://www.zhihu.com/equation?tex=s) 和 ![t](https://www.zhihu.com/equation?tex=t) 的值并不会暴露关于 ![a](https://www.zhihu.com/equation?tex=a) 和 ![b](https://www.zhihu.com/equation?tex=b) 的信息。上面那个式子中， ![s\times t](https://www.zhihu.com/equation?tex=s%5Ctimes+t) 可以直接用公开的 ![s](https://www.zhihu.com/equation?tex=s) 和 ![t](https://www.zhihu.com/equation?tex=t) 算出来， ![s \times b](https://www.zhihu.com/equation?tex=s+%5Ctimes+b) 以及 ![t \times a](https://www.zhihu.com/equation?tex=t+%5Ctimes+a) 的密钥分享则可以用前面的秘密数与公开数的乘法得到，而 ![c](https://www.zhihu.com/equation?tex=c) 的密钥分享则是一开始就存在，因此这几项合起来便能得到 ![z = x \times y](https://www.zhihu.com/equation?tex=z+%3D+x+%5Ctimes+y) 的密钥分享。

需要注意的是，每个这样的秘密数字的乘法都会消耗一组随机乘积元组，不过由于随机乘积元组的值和计算时的 ![x](https://www.zhihu.com/equation?tex=x) 和 ![y](https://www.zhihu.com/equation?tex=y) 的值是无关的，因此这样的元组可以由参与方在空闲的时候预先生成一大堆，等需要用上的时候再拿出来消耗掉。

在密钥分享中，由于每次计算后得到的仍然是密钥分享，因此各操作可以串起来，直到算到最终结果，再将其暴露出来。有了加法和乘法，我们理论上可以进行各种计算，比如除法和指数都可以用加法和乘法去拟合，浮点数运算也可以模拟，具体就不展开了。

