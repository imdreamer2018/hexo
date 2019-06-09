title: 可扩展隐私保护机器学习系统
author: 追梦人

toc: true

tags:

- MachineLearning
- Privacy-Preserving
- Cryptology
  categories: []
  date: 2019-06-09 21:23:00

---

# CodedPrivateML：分布式机器学习的快速和隐私保护框架

​			Jinhyun So *1Bas¸akGuler¨* 1 A. Salman Avestimehr 1 Payman Mohassel 2

## 摘要

如何在保持数据私密性和安全性的同时培养机器学习模型？我们提出CodedPrivateML，这是一个快速，可扩展的方法来解决这个关键问题。CodedPrivateML保持数据和模型信息 - 理论上是私有的，同时允许跨分布式工作人员的培训的高效并行化。我们描述了CodedPrivateML的隐私阈值，并证明了它在逻辑（和线性）回归方面的收敛性。此外，通过Amazon EC2上的实验，我们证明CodedPrivateML可以提供比最先进的加密方法快一个数量级的加速（高达~34倍）

<!-- more-->

## 1.介绍

现代机器学习模型通过在各种应用领域实现前所未有的性能而开辟新天地。然而，训练这样的模型是一项艰巨的任务。由于通常大量的数据和模型的复杂性，培训是计算和存储密集型任务。此外，通常应对敏感数据进行培训，例如医疗保健记录，浏览历史记录或金融交易，这会引发数据集的安全性和隐私性问题。这造成了一个具有挑战性的困境。
一方面，由于其复杂性，通常希望将培训外包给更有能力的计算平台，例如云。另一方面，训练数据集通常是敏感的，应特别注意保护数据集的隐私免受此类平台中的潜在破坏。这种困境引发了我们在这里研究的主要问题：我们如何将培训任务上传到分布式计算平台，同时保持数据集的隐私？

更具体地，我们考虑一种情况，其中数据所有者（例如，医院）希望通过上传大量数据（例如，医疗保健记录）和计算密集型训练任务（例如，梯度计算）来训练逻辑回归模型。）通过云平台到N台机器，同时确保N个工作人员中的T之间的任何共谋不泄漏有关训练数据集的信息。我们专注于半诚实的对手设置，其中受损方遵循协议但可能泄漏信息以尝试学习训练数据集。

我们为这个问题提出了CodedPrivateML，它有三个显着特征：

1. 在共谋工人面前，为训练数据集和模型参数提供强有力的信息理论隐私保障。
2. 通过在几个工人之间有效地分配训练计算负荷，实现快速训练。
3. 利用一种新方法，基于编码和信息理论原则秘密共享数据集和模型参数，从而显着降低通信开销和分布式培训的复杂性。

在较高的层次上，CodedPrivateML可以描述如下。它通过两个步骤秘密分享每轮培训的数据集和模型参数。首先，它采用随机量化将每一轮的数据集和权重向量转换为有限域。然后，它使用一种名为拉格朗日编码（Yu et al。，2019）的新型编码技术将量化值与随机矩阵组合（或编码），以保证隐私（在信息理论意义上），同时在多个工人之间分配工作量 。然而，挑战是拉格朗日编码只能用于多项式评估形式的计算。另一方面，逻辑回归的梯度计算包括不能表示为多项式的非线性。CodedPrivateML通过训练阶段中非线性S形函数的多项式近似来处理此挑战。

在秘密共享编码数据集和模型参数时，每个工作人员使用所选择的sigmoid函数的多项式近似来执行梯度计算，并将结果发送回主控器。值得注意的是，工作人员对量化和编码数据执行计算，就好像他们在真实数据集上进行计算一样。也就是说，计算的结构对于在真实数据集上的计算与在编码数据集上的计算相同。

最后，主人从最快工人的子集中收集结果，并在有限域上解码梯度。然后，它将解码的梯度转换为真实域，更新权重向量，并将秘密与工作节点共享以用于下一轮。我们注意到，由于计算是在有限域中执行的，而权重是在真实域中更新的，因此更新过程可能导致不期望的行为，因为权重可能不会收敛。我们的系统通过所提出的随机量化技术保证收敛，同时在实场和有限域之间进行转换。

我们理论上证明CodedPrivateML保证了模型参数的收敛，同时为训练数据集提供了信息理论隐私。我们的理论分析还确定了隐私和并行化之间的权衡。更具体地，通过减少每个工作人员的计算负荷，可以通过防止更大数量的共谋T或更多并行化来利用每个附加工作者以获得更多隐私。我们描述了CodedPrivateML的权衡。

此外，我们通过将CodedPrivateML与基于安全多方计算（MPC）的最先进的加密方法（Yao，1982; Ben-Or et al。1988）进行比较，经验证明了CodedPrivateML的影响。应用于启用隐私保护机器学习任务（例如参见（Nikolaenko等，2013; Gascon等人，2017; Mohassel＆Zhang，2017; Lindell＆Pinkas，2000; Dahl等，2018; Chen等） al。，2019））。特别地，我们设想秘密在多个工人之间共享其数据和模型参数的主人，他们使用多轮MPC协议共同执行梯度计算。鉴于我们对信息理论隐私的关注，最相关的基于MPC的经验比较方案是基于Shamir秘密共享的BGW式（Ben-Or等，1988）方法（Shamir，1979）。虽然最近的一些工作设计了具有信息理论安全性的基于MPC的私人学习解决方案（Wagh等，2018; Mohassel＆Rindal，2018），但它们的结构仅限于三方或四方。

我们在Amazon EC2云上进行了大量实验，以实证地展示CodedPrivateML的性能。我们通过MNIST数据集训练逻辑回归模型进行图像分类（LeCun等，2010），而计算工作量通过云分布到N = 40台机器。我们证明，与基于MPC的方案相比，CodedPrivateML可以在训练时间内提供大幅加速（高达~34.1×），同时保证相同的准确度。基于MPC的方案的主要缺点是它依赖于工作人员之间用于分布式私人计算的广泛通信和协调，并且不受益于工人之间的并行化，因为参与MPC的所有参与者都重复整个计算。然而，与CodedPrivateML相比，它们保证了更高的隐私阈值（即，更大的T）。

**其他相关工作**。除了针对该问题的基于MPC的方案之外，还可以考虑另外两种解决方案。一种是基于同态加密（HE）（Gentry＆Boneh，2009），它允许对加密数据进行计算，并且已被用于实现保护隐私的机器学习解决方案（GiladBachrach等，2016; Hesamifard等。2017; Graepel等，2012; Yuan＆Yu，2014; Li等，2017; Kim等，2018; Wang等，2018; Han等，2019）。HE的隐私保证基于计算假设，而我们的系统提供强大的信息理论安全性。此外，HE要求对加密数据执行计算，这导致训练中的许多数量级减慢。例如，对于MNIST数据集上的图像分类，HE需要2小时来学习具有％96准确度的逻辑回归模型（Han等，2019）。相反，在CodedPrivateML中，执行编码计算没有减慢速度，从而可以更快地实现。作为权衡，HE允许在更多数量的工人之间进行勾结，而在CodedPrivateML中，该数量由其他系统参数确定，例如工人数量和分配给每个工人的计算负荷。

另一种可能的解决方案是基于差异隐私（DP），这是一种保留个人身份信息隐私的发布机制，因为从数据集中删除任何单个元素不会显着改变计算结果（Dwork等，2006）。在机器学习的背景下，DP主要用于在模型参数发布供公众使用时进行训练，以确保无法从发布的模型中识别来自数据集的各个数据点（Chaudhuri＆Monteleoni，2009; Shokri）＆Shmatikov，2015; Abadi等，2016; Pathak等，2010; McMahan等，2018; Rajkumar＆Agarwal，2012; Jayaraman等，2018）。这些方法与我们的工作之间的主要区别在于，我们可以保证强大的信息理论隐私，不会泄漏有关数据集的信息，并在整个培训过程中保持模型的准确性。然而，我们注意到，如果打算公开发布最终模型，我们原则上可以使用差异隐私来编写CodedPrivateML技术以获得两全其美，但我们将此作为未来的工作。

![VshCsx.png](https://s2.ax1x.com/2019/06/09/VshCsx.png)

图1.分布式培训设置，包括主节点和N个工作节点。主设备与每个工作人员共享数据集的编码版本（由$\widetilde{\mathbf{X}}_{i}​$ 's表示）和模型参数的当前估计（由$\widetilde{\mathbf{W}}_{i}^{(t)}​$ 's表示）以保证数据集的信息理论隐私不受任何T的影响。联系工人。工作人员在编码数据上本地执行计算，并将结果发送回主数据。

## 2.问题设定

我们研究了训练逻辑回归模型的问题。训练数据集由矩阵$\mathbf{X} \in \mathbb{R}^{m \times d}​$表示，矩阵由具有$d​$个特征的$m​$个数据点和标签矢量$\mathbf{y} \in\{0,1\}^{m}​$组成。$X​$的行$i​$用$\mathbf{X}_{i}​$表示。

模型参数（权重）$\mathbf{w} \in \mathbb{R}^{d}​$是通过最小化交叉熵函数获得的.
$$
C(\mathbf{w})=\frac{1}{m} \sum_{i=1}^{m}\left(-y_{i} \log \hat{y}_{i}-\left(1-y_{i}\right) \log \left(1-\hat{y}_{i}\right)\right) \tag{1}
$$
其中$\hat{y}_{i}=g\left(\mathbf{x}_{i} \cdot \mathbf{w}\right) \in(0,1)​$是标签$i​$等于1的估计概率，$g(\cdot)​$是sigmoid函数
$$
g(z)=1 /\left(1+e^{-z}\right) \tag{2}
$$
(1)中的问题可以通过梯度下降，通过迭代过程来解决，该过程在梯度的相反方向上更新模型参数。(1)的梯度由$\nabla C(\mathbf{w})=\frac{1}{m} \mathbf{X}^{\top}(g(\mathbf{X} \times \mathbf{w})-\mathbf{y})​$给出。因此，模型参数更新为
$$
\mathbf{w}^{(t+1)}=\mathbf{w}^{(t)}-\frac{\eta}{m} \mathbf{X}^{\top}\left(g\left(\mathbf{X} \times \mathbf{w}^{(t)}\right)-\mathbf{y}\right) \tag{3}
$$
其中$\mathbf{w}^{(t)}​$保持来自迭代$t​$的估计参数，$\eta​$是学习速率，并且函数$g(\cdot)​$在由$\mathbf{X} \times \mathbf{w}^{(t)}​$给出的矢量上逐元素地操作。

如图1所示，我们考虑一个主工作者分布式计算架构，其中主服务器将计算密集型操作发布到$N​$个工作者。
这些操作对应于(3)中的梯度计算。在这样做时，主人希望保护数据集$X​$的隐私免受最多$T​$工作者之间的任何潜在共谋，其中$T​$是系统的隐私参数。

在培训开始时，数据集X在工作人员之间以隐私保护的方式共享。为此，首先将$X​$划分为$K​$个子矩阵$\mathbf{X}=\left[\mathbf{X}_{1}^{\top} \cdots \mathbf{X}_{K}^{\top}\right]^{\top}​$,对于某些$K \in \mathbb{N}​$。参数$K​$与每个工人的计算负荷相关（即，每个工人处理的数据集的比例），以及主人必须等待的工人数量，以重建梯度在每一步。然后，主设备创建N个编码的子矩阵，由$\tilde{\mathbf{X}}_{1}, \ldots, \tilde{\mathbf{X}}_{N}​$，通过将数据集的K部分与一些随机矩阵组合以保护隐私，并将$\widetilde{\mathbf{X}}_{i}​$发送给工人$i \in[N]​$。此过程应仅对数据集$X​$执行一次。

在训练的每次迭代t，主设备还需要向工人$i \in\lfloor N\rfloor​$发送模型参数的当前估计((3)中的i.e., $\mathbf{w}^{(t)}​$)。然而，最近显示，中间模型参数也可能泄漏关于数据集的大量信息（Melis等，2019）。主机还需要防止这些中间参数的泄漏。为此，主设备创建编码矩阵$\widetilde{\mathbf{w}}_{i}^{(t)}​$以秘密地与工作者$i \in[N]​$共享模型参数的当前估计。这种编码策略也应该是针对任何$T​$联结工人的私人策略。

更具体地说，用于秘密共享数据集（即创建$\widetilde{\mathbf{X}}_{i}​$）和模型参数（即创建）$\widetilde{\mathbf{W}}_{i}^{(t)}​$的编码策略应该是这样的，以致任何$T​$勾结工人的子集都不能学习任何在强信息理论意义上，关于训练数据集$X​$的信息。正式地说，对于每个工作人员$\mathcal{T} \subseteq[N]​$的大多数$T​$，我们应该有，
$$
I\left(\mathbf{X} ; \widetilde{\mathbf{X}}_{\mathcal{T}},\left\{\widetilde{\mathbf{W}}_{\mathcal{T}}^{(t)}\right\}_{t \in[J]}\right)=0 \tag{4}
$$
其中I表示互信息，$J​$是迭代次数，并且$\widetilde{\mathbf{X}}_{\mathcal{T}},\left\{\widetilde{\mathbf{W}}_{\mathcal{T}}^{(t)}\right\}_{t \in[J]}​$是存储在$\mathcal{T}​$中的工人处的编码矩阵和编码参数估计的集合。我们将协议保护作为$T​$-private协议来保证$T​$串通工作者的隐私。

在每次迭代时，工人$i \in[N]​$使用$\widetilde{\mathbf{X}}_{i}​$和$\widetilde{\mathbf{W}}_{i}^{(t)}​$在本地执行其计算，并将结果发送回主控器。在从足够数量的工人接收到结果后，主人恢复$\mathbf{X}^{\top} g\left(\mathbf{X} \times \mathbf{w}^{(t)}\right)=​$$\sum_{k=1}^{K} \mathbf{X}_{k}^{\top} g\left(\mathbf{X}_{k} \times \mathbf{w}^{(t)}\right)​$，重建梯度，并更新模型参数使用(3)。在这样做时，主人需要只等待最快的工人。我们将协议的恢复阈值定义为主服务器需要等待的最小工作数。恢复阈值与参数$N​$，$K​$和$T​$之间的关系将在我们的理论分析中详述

**备注1**.*虽然我们的演示基于逻辑回归，但CodedPrivateML也可以应用于线性回归，只需稍加修改*

## 3.拟议的CodedPrivateML策略

CodedPrivateML策略包含四个主要阶段，首先在下面的高级别描述，然后在本节的其余部分详细介绍。

**第1阶段：量化**。为了保证信息理论隐私，必须使用均匀随机矩阵在有限域$\mathbb{F}$中掩蔽数据集和权重向量，使得增加的随机性可以使每个数据点看起来同样可能。相反，训练任务的数据集和权重向量在实数域中定义。我们通过采用随机量化技术将参数从实域转换到有限域来解决这个问题，反之亦然。因此，在我们系统的第一阶段，主数据量化从真实域到整数域的数据集和权重，然后将它们嵌入到以素数$p$为模的整数域$\mathbb{F}_{p}$中。数据集$X$的量化版本由$\overline{X}$给出。另一方面，权重向量$\mathbf{w}^{(t)}$的量化由矩阵$\overline{\mathbf{W}}^{(t)}$表示，其中每列保持$\mathbf{w}^{(t)}$的独立随机量化。这种结构对于确保模型的收敛非常重要。选择参数$p$足够大以避免计算中的环绕。它的值取决于机器的位宽以及加法和乘法运算的数量。例如，在64位实现中，我们选择$p$ = 15485863（具有24位的最大素数），如我们的实验中所详述。

**阶段2：编码和秘密共享**。在第二阶段，主设备将量化数据集$\overline{X}​$划分为$K​$个子矩阵，并使用最近提出的拉格朗日编码技术（Yu et al。，2019）对其进行编码，我们将在3.2节中详细描述。然后它向工人$i \in[N]​$发送编码的子矩阵$\widetilde{\mathbf{X}}_{i} \in \mathbb{F}_{p}^{\frac{m}{K} \times d}​$。正如我们稍后将说明的那样，这种编码可确保编码矩阵不会泄漏有关真实数据集的任何信息，即使$T​$工作者勾结。此外，主人必须确保在每次迭代时发送给工人的重量估计不会泄漏有关数据集的信息。这是因为通过（3）更新的权重携带有关整个训练集的信息，并且将它们直接发送给工作人员可能会破坏隐私。为了防止这种情况，在迭代$t​$，主设备还将当前权重向量$\mathbf{w}^{(t)}​$量化为有限域，并使用拉格朗日编码再次对其进行编码。

**阶段3：多项式逼近和局部计算**。在第三阶段，每个工作人员使用其本地存储执行计算，并将结果发送回主服务器。我们注意到，工作人员对编码数据执行计算，就像他们在真实数据集上进行计算一样。也就是说，计算的结构对于在真实数据集上的计算与在编码数据集上的计算相同。一个主要挑战是拉格朗日编码是为分布式多项式计算而设计的。然而，由于$S​$形函数，训练阶段中的计算不是多项式。我们通过用选定度$r​$的多项式逼近sigmoid来克服这个问题。这允许我们根据可由每个工人本地计算的多项式来表示梯度计算。

**阶段4：解码和模型更新**。主人从最快工人的子集中收集结果，并在有限域上解码梯度。最后，master将解码后的渐变转换为真实域，更新权重向量，并将秘密与工作者共享以用于下一轮。我们接下来提供每个阶段的详细信息。CodedPrivateML的整体算法及其四个阶段中的每一个也在补充材料的附录A.1中给出。

### 3.1 量化

我们考虑数据集和权重的元素有损量化方案。为了量化数据集$\mathbf{X} \in​$$\mathbb{R}^{m \times d}​$，我们使用简单的确定性舍入技术：
$$
\operatorname{Round}(x)=\left\{\begin{array}{ll}{\lfloor x\rfloor} & {\text { if } \quad x-\lfloor x\rfloor< 0.5} \\ {\lfloor x\rfloor+ 1} & {\text { otherwise }}\end{array}\right. \tag{5}
$$
其中$\lfloor x\rfloor$是小于或等于$x​$的最大整数。我们将量化数据集定义为:
$$
\overline{\mathbf{X}} \triangleq \phi\left(\operatorname{Round}\left(2^{l_{x}} \cdot \mathbf{X}\right)\right) \tag{6}
$$
其中从(5)的舍入函数元素地应用于矩阵$X$的元素，并且$l_{x}$是控制量化损失的整数参数。函数$\phi : \mathbb{Z} \rightarrow \mathbb{F}_{p}$是定义为通过使用二进制补码表示来表示有限域中的负整数的映射,
$$
\phi(x)=\left\{\begin{array}{ll}{x} & {\text { if } x \geq 0} \\ {p+x} & {\text { if } x<0}\end{array}\right. \tag{7}
$$
注意，(6)的域是$\left[-\frac{p-1}{2^{\left(l_{x}+1\right)}}, \frac{p-1}{2^{\left(l_{x}+1\right)}}\right]$。为了避免可能导致溢出错误的环绕，素数p应足够大，即$p \geq 2^{l_{x}+1} \max \left\{\left|\mathbf{X}_{i, j}\right|\right\}+1$>。在每次迭代时，主设备还将权重向量$\mathbf{w}^{(t)}$从实域量化到有限域。这被证明是一项具有挑战性的任务，因为它应该以确保模型收敛的方式执行。我们的解决方案是受（Zhang et al。，2017; 2016）启发的量化技术。最初，我们定义了一个随机量化函数：
$$
Q\left(x ; l_{w}\right) \triangleq \phi\left(\operatorname{Round}_{s t o c}\left(2^{l_{w}} \cdot x\right)\right) \tag{8}
$$


其中$l_{w}$是一个整数参数来控制量化损失。Round$_{\text {stoc}} : \mathbb{R} \rightarrow \mathbb{Z}$是随机舍入函数:
$$
\operatorname{Round}_{s t o c}(x)=\left\{\begin{array}{ll}{\lfloor x\rfloor} & {\text { with prob. } 1-(x-\lfloor x\rfloor)} \\ {\lfloor x\rfloor+ 1} & {\text { with prob. } x-\lfloor x\rfloor}\end{array}\right.
$$
将$x$舍入为$\lfloor x\rfloor$的概率与$x$与$\lfloor x\rfloor$的接近度成比例，因此随机舍入是无偏的（即$\mathbb{E}\left[\text {Round}_{\text {stoc}}(x)\right]=x$）。

为了量化权重向量$\mathbf{w}^{(t)}$，主设备创建$r$个独立的量化矢量。
$$
\overline{\mathbf{w}}^{(t), j} \triangleq Q_{j}\left(\mathbf{w}^{(t)} ; l_{w}\right) \in \mathbb{F}_{p}^{d \times 1} \text { for } j \in[r] \tag{9}
$$
其中量化函数（8）以元素方式应用于向量$\mathbf{w}^{(t)}$，并且每个$Q_{j}(\because ;)$表示（8）的独立实现。量化向量的数量$r$等于sigmoid函数的多项式近似的程度，我们将在后面的3.3节中描述。创建$r$独立量化背后的直觉是确保使用量化权重执行的梯度计算是真实梯度的无偏估计。如第4节所述，该属性是我们模型收敛性分析的基础。参数$l_{x}$和$l_{w}$的具体值提供了舍入误差和溢出误差之间的折衷。特别是，较大的值会减少舍入误差，同时增加溢出的可能性。我们将权重向量$\mathbf{w}^{(t)}$的量化表示为:
$$
\overline{\mathbf{W}}^{(t)}=\left[\overline{\mathbf{w}}^{(t), 1} \ldots \cdot \overline{\mathbf{w}}^{(t), r}\right] \tag{10}
$$
通过以矩阵形式排列来自（9）的量化矢量。

### 3.2 编码和秘密共享

主设备首先将量化数据集$\overline{\mathbf{X}}$划分为$K$个子矩阵$\overline{\mathbf{X}}=\left[\overline{\mathbf{X}}_{1}^{\top} \ldots \overline{\mathbf{X}}_{K}^{\top}\right]^{\top}$，其中$\overline{\mathbf{X}}_{i} \in \mathbb{F}_{p}^{\frac{m}{K} \times d}$表示$i \in[K]$。它还选择K$K+T$个不同的元素$\beta_{1}, \dots, \beta_{K+T}$来自Fp。然后，它使用拉格朗日编码（Yu等，2019）来编码数据集。更具体地说，它找到多项式$u : \mathbb{F}_{p} \rightarrow \mathbb{F}_{p}^{\frac{m}{K} \times d}$，其度数最多为$K+T-1$，使得$u\left(\beta_{i}\right)=\overline{\mathbf{X}}_{i}$，对于$i \in[K]$，$u\left(\beta_{i}\right)=\mathbf{Z}_{i}$，对于$i \in\{K+1, \ldots, K+T\}$，其中从$\mathbb{E}_{p}^{\frac{m}{K}} \times d$随机均匀地选择$Z_{i}$（$Z_{i}$的作用是掩盖数据集并提供隐私以抵抗T勾结工人）。这是通过让你成为各自的拉格朗日插值多项式来实现的:
$$
\begin{aligned} u(z) \triangleq & \sum_{j \in[K]} \overline{\mathbf{X}}_{j} \cdot \prod_{k \in[K+T] \backslash\{j\}} \frac{z-\beta_{k}}{\beta_{j}-\beta_{k}} \\ &+\sum_{j=K+1}^{K+T} \mathbf{Z}_{j} \cdot \prod_{k \in[K+T] \backslash\{j\}} \frac{z-\beta_{k}}{\beta_{j}-\beta_{k}} \end{aligned} \tag{11}
$$
然后，Master从$\mathbb{F}_{p}$中选择$N$个不同的元素$\left\{\alpha_{i}\right\}_{i \in[N]}$，使$\left\{\alpha_{i}\right\}_{i \in[N]} \cap\left\{\beta_{j}\right\}_{j \in[K]}=\varnothing$，并通过让$\tilde{\mathbf{X}}_{i}=u\left(\alpha_{i}\right)$对数据集进行编码）对于$i \in[N]$。通过定义编码矩阵$\mathbf{U}=\left[\mathbf{u}_{1} \ldots \mathbf{u}_{N}\right] \in \mathbb{F}_{p}^{(K+T) \times N}$其第$(i, j)^{t h}$个元素由给$u_{i j}=\prod_{\ell \in[K+T] \backslash\{i\}} \frac{\alpha_{j}-\beta_{\ell}}{\beta_{i}-\beta_{\ell}}$出，也可以表示数据集的编码为:
$$
\tilde{\mathbf{X}}_{i}=u\left(\alpha_{i}\right)=\left(\overline{\mathbf{X}}_{1}, \ldots, \overline{\mathbf{X}}_{K}, \mathbf{Z}_{K+1}, \ldots, \mathbf{Z}_{K+T}\right) \cdot \mathbf{u}_{i} \tag{12}
$$
在迭代$t$，量化权重$\overline{\mathbf{W}}^{(t)}$也使用拉格朗日插值多项式编码，
$$
\begin{aligned} v(z) \triangleq & \sum_{j \in[K]} \overline{\mathbf{W}}^{(t)} \cdot \prod_{k \in[K+T] \backslash\{j\}} \frac{z-\beta_{k}}{\beta_{j}-\beta_{k}} \\ &+\sum_{j=K+1}^{K+T} \mathbf{V}_{j} \cdot \prod_{k \in[K+T] \backslash\{j\}} \frac{z-\beta_{k}}{\beta_{j}-\beta_{k}} \end{aligned} \tag{13}
$$
其中，$j \in[K+1, K+T]$的$\mathbf{V}_{j}$从$\mathbb{F}_{p}^{d \times r}$随机均匀地选择。系数$\beta_{1}, \ldots, \beta_{K+T}$与（11）中的相同。我们注意到（13）中的多项式对于$i \in[K]$具有$v\left(\beta_{i}\right)=\overline{\mathbf{W}}^{(t)}$ 的性质。

然后，主设备通过使用相同的评估点$\left\{\alpha_{i}\right\}_{i \in[N]}$对量化的权重向量进行编码。因此，权重向量被编码为:
$$
\widetilde{\mathbf{W}}_{i}^{(t)}=v\left(\alpha_{i}\right)=\left(\overline{\mathbf{W}}^{(t)}, \ldots, \overline{\mathbf{W}}^{(t)}, \mathbf{V}_{K+1}, \ldots, \mathbf{V}_{K+T}\right) \cdot \mathbf{u}_{i} \tag{14}
$$
对于$i \in[N]$，使用来自（12）的编码矩阵$U$.多项式$u(z)$和$v(z)$的程度都是$K+T-1$。

### 3.3 多项式逼近与局部计算

在接收到编码（和量化的）数据集和权重后，工作人员应继续进行梯度计算。然而，一个主要的挑战是拉格朗日编码最初是为多项式计算而设计的，而工人需要做的梯度计算不是由于S形函数的多项式。我们的解决方案是使用sigmoid函数的多项式近似.
$$
\hat{g}(z)=\sum_{i=0}^{r} c_{i} z^{i} \tag{15}
$$
其中$r$和$c_i$分别表示多项式的次数和系数。通过最小二乘估计拟合$S$形函数来获得系数。

使用这个多项式近似，我们可以重写（3）为：
$$
\mathbf{w}^{(t+1)}=\mathbf{w}^{(t)}-\frac{\eta}{m} \overline{\mathbf{X}}^{\top}\left(\hat{g}\left(\overline{\mathbf{X}} \times \mathbf{w}^{(t)}\right)-\mathbf{y}\right) \tag{16}
$$
其中$\overline{\mathbf{X}}$是$X$的量化版本，并且$\hat{g}(\cdot)$在向量$\overline{\mathbf{X}} \times \mathbf{w}^{(t)}$上以元素方式操作。

另一个挑战是确保权重的收敛。正如我们在第4节中详述的那样，这需要使用具有量化权重的多项式近似来无偏差地进行梯度估计。我们利用第3.1节中形成的量化权重利用Lemma 4.1（Zhang et al。，2016）中的计算技术来解决这个问题。具体来说，给定来自（15）的$r$次多项式和来自（10）的$r$个独立量化，我们定义一个函数， 
$$
\overline{g}\left(\overline{\mathbf{X}}, \overline{\mathbf{W}}^{(t)}\right) \triangleq \sum_{i=0}^{r} c_{i} \prod_{j \leq i}\left(\overline{\mathbf{X}} \times \overline{\mathbf{w}}^{(t), j}\right) \tag{17}
$$
其中乘积$\prod_{j \leq i}$对于$j \leq i$在矢量$\left(\overline{\mathbf{X}} \times \overline{\mathbf{w}}^{(t), j}\right)$上逐个元素地操作。最后，我们注意到（17）是$\hat{g}\left(\overline{\mathbf{X}} \times \mathbf{w}^{(t)}\right)$的无偏估计，
$$
E\left[\overline{g}\left(\overline{\mathbf{X}}, \overline{\mathbf{W}}^{(t)}\right)\right]=\hat{g}\left(\overline{\mathbf{X}} \times \mathbf{w}^{(t)}\right) \tag{18}
$$
其中$\hat{g}(\cdot)$在向量$\overline{\mathbf{X}} \times \mathbf{w}^{(t)}$上以元素方式起作用，结果来自量化的独立性。

使用（17），我们根据量化权重重写（16）中的更新方程，
$$
\mathbf{w}^{(t+1)}=\mathbf{w}^{(t)}-\frac{\eta}{m} \overline{\mathbf{X}}^{\top}\left(\overline{g}\left(\overline{\mathbf{X}}, \overline{\mathbf{W}}^{(t)}\right)-\mathbf{y}\right) \tag{19}
$$
然后在本地每个工作人员执行计算。特别是，在每次迭代时，工人$i \in[N]$局部计算$f : \mathbb{F}_{p}^{\frac{m}{K} \times d} \times \mathbb{F}_{p}^{d \times r} \rightarrow \mathbb{F}_{p}^{d}$：
$$
f\left(\widetilde{\mathbf{X}}_{i}, \widetilde{\mathbf{W}}_{i}^{(t)}\right)=\widetilde{\mathbf{X}}_{i}^{\top} \overline{g}\left(\tilde{\mathbf{X}}_{i}, \widetilde{\mathbf{W}}_{i}^{(t)}\right) \tag{20}
$$
使用$\widetilde{\mathbf{X}}_{i}$和$\widetilde{\mathbf{W}}_{i}^{(t)}$并将结果发送回主站。该计算是有限域算术中的多项式函数评估，并且$f$的度数是$\operatorname{deg}(f)=2 r+1$ 。

### 3.4 解码和模型更新

在从足够数量的工人接收到（20）中的评估结果之后，主设备在有限区域上解码$\left\{f\left(\overline{\mathbf{X}}_{k}, \overline{\mathbf{W}}^{(t)}\right)\right\}_{k \in[K]}$。

主人需要等待的最小工人数被称为系统的恢复阈值，并且等于$(2 r+1)(K+T-1)+1$，如我们在Section中所示。我们现在继续解码的细节。通过构造（11）和（13）中的拉格朗日多项式，可以定义单变量多项式$h(z)=f(u(z), v(z))$这样的，
$$
\begin{aligned} h\left(\beta_{i}\right) &=f\left(u\left(\beta_{i}\right), v\left(\beta_{i}\right)\right) \\ &=f\left(\overline{\mathbf{X}}_{i}, \overline{\mathbf{W}}^{(t)}\right)=\overline{\mathbf{X}}_{i}^{\top} \overline{g}\left(\overline{\mathbf{X}}_{i}, \overline{\mathbf{W}}^{(t)}\right) \end{aligned} \tag{21}
$$
对于$i \in[K]$。另一方面，从（20），工人$i$的计算结果等于:
$$
\begin{aligned} h\left(\alpha_{i}\right) &=f\left(u\left(\alpha_{i}\right), v\left(\alpha_{i}\right)\right) \\ &=f\left(\widetilde{\mathbf{X}}_{i}, \widetilde{\mathbf{W}}_{i}^{(t)}\right)=\widetilde{\mathbf{X}}_{i}^{\top} \overline{g}\left(\widetilde{\mathbf{X}}_{i}, \widetilde{\mathbf{W}}_{i}^{(t)}\right) \end{aligned} \tag{22}
$$
（22）解码过程背后的主要直觉是使用来自（22）的计算作为评估点$h\left(\alpha_{i}\right)$来内插多项式$h(z)$。具体来说，主人可以从$(2 r+1)(K+T-1)+1$获得$h(z)$的所有系数评估结果，只要$\operatorname{deg}(h(z)) \leq(2 r+1)(K+T-1$)。恢复$h(z)$后，为了恢复恢复（21），主人可以通过当$i \in[K]$计算$h\left(\beta_{i}\right)$和评估，
$$
\sum_{k=1}^{K} f\left(\overline{\mathbf{X}}_{k}, \overline{\mathbf{W}}^{(t)}\right)=\sum_{k=1}^{K} \overline{\mathbf{X}}_{k}^{\top} \overline{g}\left(\overline{\mathbf{X}}_{k}, \overline{\mathbf{W}}^{(t)}\right)=\overline{\mathbf{X}}^{\top} \overline{g}\left(\overline{\mathbf{X}}, \overline{\mathbf{W}}^{(t)}\right) \tag{23}
$$
最后，主设备将（23）从有限域转换到真实域并根据（19）更新权重。这种转换是由$f$实现的，
$$
Q_{p}^{-1}(\overline{x} ; l)=2^{-l} \cdot \phi^{-1}(\overline{x}) \tag{24}
$$
我们让$l=l_{x}+r\left(l_{x}+l_{w}\right)$，并且$\phi^{-1} : \mathbb{F}_{p} \rightarrow \mathbb{R}$定义为，
$$
\phi^{-1}(\overline{x})=\left\{\begin{array}{lll}{\overline{x}} & {\text { if }} & {0 \leq \overline{x}<\frac{p-1}{2}} \\ {\overline{x}-p} & {\text { if }} & {\frac{p-1}{2} \leq \overline{x}<p}\end{array}\right. \tag{25}
$$

## 4 融合和隐私保障

考虑成本函数（1），我们的目标是在使用（6）用量化数据集$\overline{\mathbf{X}}$替换数据集时$\mathbf{X}$在逻辑回归中最小化。同时将$\mathbf{W}^{*}$表示为最小化的最佳权重向量（1）当$\hat{y}_{i}=g\left(\overline{\mathbf{x}}_{i} \cdot \mathbf{w}\right)$时，其中$\overline{\mathbf{X}}_{i}$是$\overline{\mathbf{X}}$的第$i$行。在本节中，我们证明CodedPrivateML将保证收敛到最优模型参数（即，$\mathbf{W}^{*}$）同时保持数据集的隐私，防止勾结工人。

回想一下，CodedPrivateML中主节点的模型更新遵循（19），即
$$
\mathbf{w}^{(t+1)}=\mathbf{w}^{(t)}-\frac{\eta}{m} \overline{\mathbf{X}}^{\top}\left(\overline{g}\left(\overline{\mathbf{X}}, \overline{\mathbf{W}}^{(t)}\right)-\mathbf{y}\right) \tag{26}
$$
我们首先说明了一个引理，附录A.2在补充材料中对此进行了证明。

**引理1**，设$\mathbf{p}^{(t)} \triangleq \frac{1}{m} \overline{\mathbf{X}}^{\top}\left(\overline{g}\left(\overline{\mathbf{X}}, \overline{\mathbf{W}}^{(t)}\right)-\mathbf{y}\right)$表示使用$CodedPrivateML$中的量化权重$\overline{\mathbf{W}}^{(t)}$梯度计算。然后我们有，

- （无偏性）向量$\mathbf{p}^{(t)}$是真实梯度的渐近无偏估计。$\mathbb{E}\left[\mathbf{p}^{(t)}\right]=\nabla C\left(\mathbf{w}^{(t)}\right)+\epsilon(r)$为$r \rightarrow \infty$其中$\epsilon(r) \rightarrow 0$ 是（15）中的多项式次数，并且相对于期望采取量化误差，
- （方差界）$\mathbb{E}\left[\left\|\mathbf{p}^{(t)}-\mathbb{E}\left[\mathbf{p}^{(t)}\right]\right\|_{2}^{2}\right] \leq\frac{1}{2^{-2 l} w m^{2}}\|\overline{\mathbf{X}}\|_{F}^{2} \triangleq \sigma^{2}$其中$\|\cdot\|_{2}$ 和$\|\cdot\|_{F}$表示$l_{2}$范数和Frobenius规范分别。

我们还需要以下基本引理，补充材料的附录A.3中对此进行了证明。

**引理2**，成本函数（1）的梯度与量化数据集$\overline{\mathbf{X}}$（如（6）中定义的）是$L-Lipschitz$，$L \triangleq \frac{1}{4}\|\overline{\mathbf{X}}\|_{2}^{2}$，即对于所有$\mathbf{w}, \mathbf{w}^{\prime} \in \mathbb{R}^{d}$我们有
$$
\left\|\nabla C(\mathbf{w})-\nabla C\left(\mathbf{w}^{\prime}\right)\right\| \leq L\left\|\mathbf{w}-\mathbf{w}^{\prime}\right\| \tag{27}
$$
我们现在陈述CodedPrivateML的主要定理。

**定理1**。考虑在分布式系统中使用$CodedPrivateML$，数据集$\mathbf{X}=\left(\mathbf{X}_{1}, \dots, \mathbf{X}_{K}\right)$，初始权重向量$\mathbf{w}^{(0)}$和常数步长$\eta=1 / L$的分布式系统中逻辑回归模型的训练（其中$L$在引理2中定义）。然后，$CodedPrivateML$保证，

- （收敛）$\mathbb{E}\left[C\left(\frac{1}{J} \sum_{t=0}^{J} \mathbf{w}^{(t)}\right)\right]-C\left(\mathbf{w}^{*}\right) \leq\frac{\left\|\mathbf{w}^{(0)}-\mathbf{w}^{*}\right\|^{2}}{2 \eta J}+\eta \sigma^{2}$在$J$次迭代中，其中$\sigma^{2}$是在引理1中给出，
- （隐私）$X$在理论上仍然是任何$T$串通工人的私人信息，即$I\left(\mathbf{X} ; \tilde{\mathbf{X}}_{\mathcal{T}},\left\{\widetilde{\mathbf{W}}_{\mathcal{T}}^{(t)}\right\}_{t \in[J]}\right)=0, \forall \mathcal{T} \subset[N],|\mathcal{T}| \leq T​$

只要我们有$N \geq(2 r+1)(K+T-1)+1​$，其中r是（15）中多项式近似的次数。

**备注2**。定理1揭示了$CodedPrivateML$中隐私和并行化之间的重要权衡。参数$K$反映了$CodedPrivateML$中的并行化量，因为每个工作节点的计算负载与数据集的$1 / K-th$成比例。参数$T$还反映了$CodedPrivateML$中的隐私阈值。定理1表明，在具有$N$个工人的集群中，只要$N \geq(2 r+1)(K+T-1)+1$，我们就可以实现任何$K$和$T$.这条件进一步暗示，作为工人的数量$N$增加，$CodedPrivateML$的并行化（$K$）和隐私阈值（$T$）也可以线性增加，从而产生可扩展的解决方案。

**备注3**。定理1也适用于更简单的线性回归问题。证据背后的步骤相同。

**证明**。（收敛）首先，我们证明只要$N \geq(2 r+1)(K+T-1)+1$，主设备就可以在有限字段上解码$\overline{\mathbf{X}}^{\top} \overline{g}\left(\overline{\mathbf{X}}, \overline{\mathbf{W}}^{(t)}\right)$。如上所述在第3.3和3.4节中，给定（15）中的S形函数近似的多项式，（21）中的$h(z)$的程度最大$(2 r+1)(K+T-1)$。解码过程使用来自工人的计算作为评估点$h\left(\alpha_{i}\right)$$h(z)$来内插多项式h（z）。主人可以获得$h(z)$的所有系数，只要主人收集至少deg$(h(z))+1 \leq(2 r+1)(K+T-1)+1$评估结果为$h\left(\alpha_{i}\right)$$h(z)$。在恢复$h(z)$之后，主设备可以通过计算$h\left(\beta_{i}\right)$ ， $i \in[K]$来解码子梯度$\overline{\mathbf{X}}_{i}^{\top} \overline{g}\left(\overline{\mathbf{X}}_{i}, \overline{\mathbf{W}}^{(t)}\right)$。因此，恢复阈值由$(2 r+1)(K+T-1)+1$给出以解码$\overline{\mathbf{X}}_{i}^{\top} \overline{g}\left(\overline{\mathbf{X}}_{i}, \overline{\mathbf{W}}^{(t)}\right)$。

接下来，我们考虑$CodedPrivateML$中的更新方程（见（26））并证明其收敛到$\mathbf{W}^{*}$。从引理2中所述的$\nabla C(\mathbf{w})$的$L-Lipschitz$连续性，我们得到了，
$$
\begin{aligned} C\left(\mathbf{w}^{(t+1)}\right) & \leq C\left(\mathbf{w}^{(t)}\right)+\left\langle\nabla C\left(\mathbf{w}^{(t)}\right), \mathbf{w}^{(t+1)}-\mathbf{w}^{(t)}\right\rangle \\ &+\frac{L}{2}\left\|\mathbf{w}^{(t+1)}-\mathbf{w}^{(t)}\right\|^{2} \\ &=C\left(\mathbf{w}^{(t)}\right)-\eta\left\langle\nabla \mathrm{C}\left(\mathbf{w}^{(t)}\right), \mathbf{p}^{(t)}\right\rangle+\frac{L}{2}\left\|\mathbf{p}^{(t)}\right\|^{2} \end{aligned}
$$
其中$\langle, \cdot,\rangle$是内在产品。通过对双方量化噪声的期望，
$$
\begin{array}{l}{\mathbb{E}\left[C\left(\mathbf{w}^{(t+1)}\right)\right]} \\ { \leq C\left(\mathbf{w}^{(t)}\right)-\eta\left\|\nabla C\left(\mathbf{w}^{(t)}\right)\right\|^{2}+\frac{L \eta^{2}}{2}\left(\left\|\nabla C\left(\mathbf{w}^{(t)}\right)\right\|^{2}+\sigma^{2}\right)} \\ { \leq C\left(\mathbf{w}^{(t)}\right)-\eta(1-L \eta / 2)\left\|\nabla C\left(\mathbf{w}^{(t)}\right)\right\|^{2}+L \eta^{2} \sigma^{2} / 2} \\ { \leq C\left(\mathbf{w}^{(t)}\right)-\eta / 2\left\|\nabla \mathrm{C}\left(\mathbf{w}^{(t)}\right)\right\|^{2}+\eta \sigma^{2} / 2}
C\left(\mathbf{w}^{*}\right)+\left\langle\nabla \mathrm{C}\left(\mathbf{w}^{(t)}\right), \mathbf{w}^{(t)}-\mathbf{w}^{*}\right\rangle \\ -\frac{\eta}{2}\left\|\nabla \mathrm{C}\left(\mathbf{w}^{(t)}\right)\right\|^{2}+\eta \sigma^{2} / 2
\begin{array}{l}{ \leq C\left(\mathbf{w}^{*}\right)+\left\langle\mathbb{E}\left[\mathbf{p}^{(t)}\right], \mathbf{w}^{(t)}-\mathbf{w}^{*}\right\rangle-\frac{\eta}{2} \mathbb{E}\left[ \| \mathbf{p}^{(t)}\right) \|^{2} ]+\eta \sigma^{2}} \\ {=C\left(\mathbf{w}^{*}\right)+\eta \sigma^{2}+\mathbb{E}\left[\left\langle\mathbf{p}^{(t)}, \mathbf{w}^{(t)}-\mathbf{w}^{*}\right\rangle-\frac{\eta}{2} \| \mathbf{p}^{(t)}\right) \|^{2} ]} \\ {=C\left(\mathbf{w}^{*}\right)+\eta \sigma^{2}+\frac{1}{2 \eta}\left(\left\|\mathbf{w}^{(t)}-\mathbf{w}^{*}\right\|^{2}-\| \mathbf{w}^{(t+1)}-\mathbf{w}^{*}\right) \|^{2}}\end{array}
\end{array}
\tag{28}
$$
其中（28）来自$L \eta \leq 1$，（29）来自$C$的凸性，并且（30）成立，因为$\mathbb{E}\left[\mathbf{p}^{(t)}\right]=\nabla \mathrm{C}\left(\mathbf{w}^{(t)}\right)$和Ep（t））假设任意大的r，引理1中的$\mathbb{E}\left[ \| \mathbf{p}^{(t)}\right) \|^{2} ]-\left\|\nabla C\left(\mathbf{w}^{(t)}\right)\right\|^{2} \leq \sigma^{2}$。对$t=0, \dots, J-1$的上述等式求和，我们有,
$$
\begin{aligned} \sum_{t=0}^{J-1} &\left(\mathbb{E}\left[C\left(\mathbf{w}^{(t+1)}\right)\right]-C\left(\mathbf{w}^{*}\right)\right) \\ & \leq \frac{1}{2 \eta}\left(\left\|\mathbf{w}^{(0)}-\mathbf{w}^{*}\right\|^{2}-\| \mathbf{w}^{(J)}-\mathbf{w}^{*}\right) \|^{2} )+J \eta \sigma^{2} \\ & \leq \frac{\left\|\mathbf{w}^{(0)}-\mathbf{w}^{*}\right\|^{2}}{2 \eta}+J \eta \sigma^{2} \end{aligned}
$$
最后，由于$C$是凸的，我们观察到
$$
\begin{aligned} \mathbb{E}\left[C\left(\frac{1}{J} \sum_{t=0}^{J} \mathbf{w}^{(t)}\right)\right] & \leq \sum_{t=0}^{J-1}\left(\mathbb{E}\left[C\left(\mathbf{w}^{(t+1)}\right)\right]-C\left(\mathbf{w}^{*}\right)\right) \\ & \leq \frac{\left\|\mathbf{w}^{(0)}-\mathbf{w}^{*}\right\|^{2}}{2 \eta J}+\eta \sigma^{2} \end{aligned}
$$
这完成了收敛证明。
（隐私）ProofofT-privacy在附录材料中推迟到附录A.4。

## 5 实验

我们现在通过实验证明$CodedPrivateML$的影响，并与现有的问题加密方法进行比较。我们的重点是培训图像分类的逻辑回归模型，而计算负载则分布在Amazon EC2云平台上的多台机器上。

**建立**。我们从（1）中训练逻辑回归模型，用于**MNIST**数据集上的二值图像分类（LeCun等，2010），以实验检验两个方面：$CodedPrivateML$的准确性和训练时间方面的性能增益。数据集的大小为$（m，d）=（12396,1568）$。附加数据集大小的实验在补充材料的附录A.6中提供。

我们使用Python上的MPI4Py（Dalc'ın等，2005）消息传递接口实现$CodedPrivateML$。使用m3.xlarge机器实例在Amazon EC2集群上以分布式方式执行计算。

然后，我们将$CodedPrivateML$与基于$MPC$的方法进行比较，以应用于我们的问题。
特别是，我们实施了基于Shamir的秘密共享方案（Shamir，1979）的BGW式结构（Ben-Or et al。，1988），我们秘密分享N个工作人员之间的数据集，他们使用多周期协议来计算梯度 。
我们进一步结合了这里介绍的量化和近似技术，因为BGW式协议也受限于有限域上的算术运算。
有关其他详细信息，请参阅补充材料的附录A.5。

为了拥有更大的数据集，我们复制了MNIST数据集。

![VshFeK.png](https://s2.ax1x.com/2019/06/09/VshFeK.png)

图2. $CodedPrivateML$相对于基于$MPC$的方案的性能增益。该图显示了Amazon EC2云平台中不同工作人员数N的准确性总培训时间95.04％（25次迭代）。

表1.总运行时间的细分，N = 40个工人。

|           协议            | 加密时间 | 通讯时间 | 比较时间 | 总运行时间 |
| :-----------------------: | :------: | -------- | -------- | ---------- |
|      $MPC approach$       |  845.55  | 49.51    | 3457.99  | 4304.60    |
| $CodedPrivateML (Case 1)$ |  50.97   | 3.01     | 66.95    | 126.20     |
| $CodedPrivateML (Case 2)$ |  90.65   | 6.45     | 110.97   | 222.50     |

**CodedPrivateML参数**。$CodedPrivateML$中有几个系统参数应该设置。鉴于我们有64位实现，我们选择字段大小为$p = 15485863$，这是24位的最大素数，以避免中间乘法的溢出。然后我们通过考虑舍入和过流错误之间的权衡来优化量化参数，在（6）中的$l_x$和在（9）中的$l_w$。特别是，我们选择$l_x = 2$和$l_w = 4$。我们还需要设置参数r，即用于近似sigmoid函数的多项式的次数。我们考虑$r = 1$和$r = 2$，并且正如我们稍后在经验上观察到的，一级近似提供了非常好的准确性。我们最终需要在$CodedPrivateML$中选择$T$（隐私阈值）和$K$（并行化量）。如定理1所述，这些参数应满足$N \geq(2 r+1)(K+T-1)+1$.鉴于我们选择$r = 1$，我们考虑两种情况：

- **案例1（最大并行化）**。通过设置$K=\left\lfloor\frac{N-1}{3}\right\rfloor$ 和 $ T=1$来并行化所有资源。
- **案例2（相等的并行化和隐私）**。通过设置$K=T=\left\lfloor\frac{N+2}{6}\right\rfloor​$来平均分配资源。

![VshPL6.png](https://s2.ax1x.com/2019/06/09/VshPL6.png)

图3. $CodedPrivateML$（针对案例2和$N = 40$工作人员演示）与使用sigmoid函数而不进行量化的常规逻辑回归的准确性的比较。使用针对3到7之间的二元分类问题重构的MNIST数据集来测量准确度（对于训练集使用12396个样本，对于测试集使用2038个样本）。

训练时间。在第一组实验中，我们测量训练时间，同时逐渐增加工人数量。结果如图2所示。我们进行了以下观察。

- $CodedPrivateML$提供了超过$MPC$方法的大幅加速，特别是在案例1和案例2中分别高达34.1倍和19.4倍加速。
  表1中显示了一个场景的总运行时间细分。可以注意到，$CodedPrivateML$在所有三类数据集编码和秘密共享方面都提供了显着的改进。工人与主人之间的沟通时间和计算时间。其中一个原因是，在基于$MPC$的方案中，每个工作人员的秘密共享数据集的大小与原始数据集相同，而在$CodedPrivateML$中，它是数据集的$1 / K$.这为$CodedPrivateML$提供了大的并行化增益。另一个原因是基于$MPC$的方案的通信复杂性。我们在补充材料的附录A.6中提供了更多场景的结果。
- 我们注意到$CodedPrivateML​$的总运行时间随着工作人员数量的增加而减少。这再次归因于$CodedPrivateML​$的并行化增益（即，当N增加时增加K）。在基于$MPC​$的方案中，这种并行化增益是不可实现的，因为所有参与$MPC​$的玩家都必须重复整个计算。然而，我们应该指出基于$MPC​$的方案可以获得更高的隐私阈值$(T=N / 2-1)​$，而$CodedPrivateML​$可以实现$T=\left\lfloor\frac{N+2}{6}\right\rfloor​$（情况2）。

**准确性**。我们还研究了$CodedPrivateML$在实验中的准确性和收敛性。图3说明了数字3和7之间二进制分类问题的测试精度。对于25次迭代，具有一次多项式逼近和常规逻辑回归的$CodedPrivateML$的准确度分别为95.04％和95.98％。该结果表明$CodedPrivateML$保证几乎相同的准确度，同时保护隐私。我们的实验还表明，$CodedPrivateML$以与传统逻辑回归相当的速率实现了收敛。这些结果见补充材料的附录A.6。



