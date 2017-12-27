# 基于加权熵对深度神经网络进行量化的方法

[原文链接](http://openaccess.thecvf.com/content_cvpr_2017/papers/Park_Weighted-Entropy-Based_Quantization_for_CVPR_2017_paper.pdf)

作者：

- Eunhyeok Park(canusglow@gmail.com)
- Junwhan Ahn § (junwhan@snu.ac.kr)
- Sungjoo Yoo (sungjoo.yoo@gmail.com)
- 首尔国立大学：计算和内存架构实验室 和 §自动化设计实验室



## 摘要

当使用嵌入式或移动设备进行开发时，由于硬件资源的限制，将原始神经网络进行量化(优化前向预测的开销)成为一种高效的优化手段。量化时需要对其进度损失进行严格的控制。本文旨在提供一种新颖的量化权重和激活输出的方式(基于加权熵)。与二值神经网络不通，我们的方法是进行多位(bit)量化，也就是将权重和激活输出可以被量化到对应位数的数字，从而达到对应的精度。这有利于更灵活地利用不同级别的量化，达到精度和性能的折衷。此外，我们的方案提供了一种基于传统训练算法的自动化量化流程，从而节省了为量化网络所花费的时间。根据对实际神经网络模型的评估：图像分类(*AlexNet*, *GoogLeNet* 和 *ResNet-50/101*)，物体检测(基于*ResNet-50*的*R-FCN*)和语言建模(一种 *LSTM* 网络)，我们的方法在损失很小精度的前提下，对模型大小和计算量都有大幅度的消减。同样，与目前市面上已经存在的量化方式相比，我们的方式在工作量更低的前提下，具有更高的精度和相同的资源约束。



## 1. 方案介绍

目前(2017年)，深度神经网络在移动和嵌入式端越来越火。由于这类系统的特点是硬件资源紧俏(能耗和内存容量)，导致程序的能会受到很大的影响。基于目前这种情况，比较典型的应用方式是，DNN前向预测过程在移动设备上进行，而DNN前向预测所使用到网络会使用服务器进行训练。因此，减少前向预测消耗对于DNN应用来说非常有必要，尤其是在移动和嵌入式系统上。

减少前向消耗的一种方式就是牺牲计算精度。虽然原始模型中的数值使用的是32/64位浮点表示，但目前研究已经论证，将原始浮点数据转换成8位数据也可以精确的进行DNN的前向预测，更有甚者转换到更窄的位宽来表示权重和激活输出值，并进行前向预测过程。另外，目前相关的研究也很活跃，旨在通过量化权重和/或激活输出，进一步降低计算量和精度。很多的量化方案，通过使用专用的硬件加速器来保证其能大幅的减小前向预测运行的时间、功耗和内存需求；例如NVIDIA的P40和P4，其支持8位整数算法或*位级串行深度神经网络计算*(其提供了与位宽成比例的执行时间和能量消耗)。

不过，现存的量化技术具的两个限制，可能会阻碍其在移动和嵌入式系统中的实际应用程序。

首先，现有的方案缺少对输出质量和前向推理的折中。移动和嵌入式系统对资源和预测精度具有严格的要求，这就需要对输出质量和预测性能进行权衡。不过，目前存在的方案都不够灵活，无法利用这种关系。例如，二值权重(深度网络)技术就会有预测输出质量损失很大的问题，这种方案就无法使用到需要很少进度损失的系统上。

其次，即使现有的量化技术能提供这种权衡，但是这些技术都需要对目标网络进行修改，以及/或支队网络的部分进行量化，从而获得不错的量化质量。由此，这种使用这种技术可能需要花费大量的设计时间，由于使用开销比较大，相关技术也很难得到推广。另外一些技术，例如对XNOR-Net和DoReFa-Net的第一层和最后一层不进行量化，避免有太大的精度损失，这可能对限制精度下降有所帮助。

为了解决这两种限制，我们采用了一种新方案——基于权重上的量化方式。我们的方法在解决了上述的两个问题的同时，量化了权重和激活输出。我们做的事情可以总结为如下几点：

1. 我们提供了一种新的多位量化的方案。与二值量化方案不同，我们的方案能够为网络提供任意位的量化，从而实现对精度-性能进行更加灵活的权衡。
2. 我们的方案有助于自动化对整个神经网络进行量化。除了激活量化之外，不需要对网络进行任何修改，因此很容易地集成到神经网络的传统训练算法中。
3. 我们用各种实际的神经网络论证了方法的效果，包括AlexNet, GoogLeNet, ResNet50/101, R-FCN和一种语言建模的LSTM网络。





## 2. 相关工作

本节，我们简单的回顾一下DNN预测的量化方式。 Vanhoucke等人对8位和32位神经网络进行过比较。 Miyashita等人建议对权重和激活输出使用以2为底的算法进行量化(称为*LogQuant*算法)，并且展示对AlexNet使用4位权重量化和5位激活输出量化的结果，其精度损失大概只有1.7%左右。使用底数量化，将模型量化到比8位更窄的数值上更具有潜力，不过AlexNet在进行低于4位的量化时，精度却有很大的损失。

目前有很多方法能将权重和/或激活输出量化到2或3级。Hwang和Sung展示了3级权重(例如：-1, 0和+1)和3位激活输出可以保证字符和音素识别任务的准确性。Courbariaux等人展示了一种二值权重网络，名为*BinaryConnect*，并证明其对小型网络(例如CIFAR-10和SVHN)有较好的精度。Rastegari等人展示了一种二值网络(二值权重版的XNOR-Net)，在AlexNet上进行实验时，没有精度的损失。

Zhou等人创造了DoReFa-Net，其对已量化的权重和有界的激活输出使用了线性量化的方式对AlexNet进行量化，权重使用1位，激活输出使用2位，与top-1的精度相差6%。他们也展示了权重使用1位，激活输出使用k位的结果，这种方式允许我们利用精度损失和性能/能耗/模型尺寸的关系进行权衡。

在之前的章节中，我们了解到这种量化方式的最大优点有两个：

1. 能灵活的使用多位的量化与精度-性能权衡和选择。
2. 量化过程无需对整个网络和现有的工作进行修改。

在这方面，大多数二级或三级权重方式都无法提供具体的方程。虽然XNOR-Net和DoReFa-Net提供了方程，但是这个方程却无法实现。XNOR-Net需要对通道进行缩放，以及使用归一化的方式重排网络层的顺序，以及将激活层放在卷积层前面。DoReFa-Net位现有网络添加了有界激活的方法。对XNOR-Net和DoReFa-Net的量化方式都不会对第一层和最后一层进行量化，这是为了避免明显的精度损失。这种限制导致要在修改网络上花费很多的时间，全精度层上具有很大性能-功耗开销，并且为了更快的对这些层进行计算，需要额外的硬件单元来支持大规模的全精度层。



## 3. 研究动机

最近的研究表明，卷积或全连接层的权重都集中的0左右，结果曲线为钟形分布。激活输出的分布曲线也类似，除了在经过ReLU层之后将激活输出的值都置为非负值。已知的量化方法也会根据这种特点决定量化的级别。例如，可以为0值尝试多种量化级数，从而使用基于对数的量化(或称*LogQuant*)可得到权值在0附近的密度分布。

除了权重或激活值的分布，我们也观察到一个很重要的信息：*量化过程中也需要考虑每个权重/激活输出值对最终结果的影响*。由于量化方法的目标是以最小的量化级别使精度损失最小化，所以考虑量化每个值的实际影响，那就需要我们开发出新方案，并更有效地的使用每个量化级别。进一步来说，我们的观点可以总结为如下几点：

1. **0附近的值**在权重和激活输出数据的分布中占据了主导地位；不过，其对输出来说影响极小(例如：权重比较小的错误值，对于卷积结果的影响是非常的小)。因此，这些值期望得到更小的量化级别(总之，*levels* 这个概念，贯穿全文)，而非通过传统的基于线性或对数的方式进行量化。
2. **大量的权重和激活输出**对于输出质量具有很大的影响，不过具有大量的权重和激活输出的网络也比较罕见。因此，为了使这些值最大化的利用，这里也期望使用较小的量化级数进行。
3. **不属于上面描述的值**中的很大部分会对输出质量有影响。因此，与传统方法相比，可为这些值分配更高的量化级别。


图1描述了现存方法为给定权重分布分配级数。

![](images/original_weight_distribution.png)

![](images/lineaer_quantization.png)

![](images/log_quantization.png)

![](images/weighted_quantization.png)

> 图1 比较各种量化方法。其中权重值是从GoogLeNet的第二个$3\times3$ 卷积层获取。每种量化方式都将该分布量化到24级。我们使用$2^{0.5}$作为对数量化的底， 并对线性和对数量化进行优化，尽可能最小化L2归一化对整体激活输出的影响。

诚然，线性量化不会考虑权重分布，对数量化会对近似为0的数给予太多的量化级数，我们的方法得到的分布更集中于那些既不太小也不太大的值。通过对量化的评估，我们将会在后面展示这种比别于传统方法效率更高的方法。



## 4. 基于加权熵的量化

### 4.1 权值量化

我们量化方法的思想是，通过某种方法将权重分成N个集群，这样就会有更多的集群分布在权重较为重要范围内，为每一个集群分配一个值，并且将每个集群中的权重量化到这个表示值内。为此，我们需要评估集群的质量，以及找到一组集群(由多个集群组成)使用该质量进行优化。

第一步，我们定义一个量化度量来决定单个权重的重要性(或影响权重的输出质量)。由于较大的权重对于输出质量有较大的影响，我们根据经验定义了重要性$i_{(n, m)}$，其为第n个集群的第m个权重的重要性。例如，$w_{(n, m)}$与其重要性的大小成一定的比例，比如，$i_{(n,m)} = w_{(n,m)}^2$。

基于每个权重的重要性，我们推导出一种用于评估集群结果(基于加权熵)质量的方式(例如：量化结果)。加权熵源于物理学中熵的概念，旨在考虑数据的重要性。

对于一组集群$C_0, ..., C_{N-1}$，加权熵S可定义为：

(1) $S = -\sum_nI_nP_nlogP_n$

这里：

(2) $P_n = \frac{\left| C_n \right|}{\sum_k\left| C_k \right|}$ (相对频率)

(3) $I_n = \frac{\sum_mi_{(n,m)}}{\left| C_n \right|}$ (代表重要性)

在这个等式中，$P_n$表示有多少个权重在集群$C_n$的范围内，并且$I_n$代表$C_n$集群中所有权重的平均重要性。大致的说，集群组中具有大量的权重，将会生成很高的的$I_n$，不过$P_n$非常小(高重要性的频率低)。根据我们的经验，通过找到集群组的最大S值，其量级会稀疏的分布在特别大的值或特别小的值附近，就如图1所示。因此，我们认为我们的权重量化有如下问题：

**问题1：权重量化**

给定训练数据(例如：迷你批量的输入)和所需要的N位对数精度(例如：N为集群的数量)，我们的方法旨在找到N个权重熵最大的N个权重集群。群集的代表值对应于权重量化中的级别。

我们使用算法1来解决这个问题：

----

**算法1：权重量化**

**function** OptSearch(N, w)

​	**for** k = 0 to $N_w$ - 1 do

​		$i_k\leftarrow f_i(w_k)$

​	$s \leftarrow sort([i_0,...,i_{N_w -1}] )$

​	$ c_0, ..., c_N \leftarrow$ initial cluster boundary

​	**while** S is increased **do**

​		**for** k = 1 to $N - 1$ **do**

​			**for** $c_k'  \in [c_{k-1}, c_{k+1}]$ **do**

​				$S' \leftarrow$ S with $c_0, ..., c'_k, ..., c_N$ 

​				**if** $S' > S$ **then**

​					$c_k \leftarrow c'_k$ 

​	**for** k = 0 to $N - 1$ **do**

​		$I_k \leftarrow \sum_{i=c_k}^{c_{k+1}-1}s[i]/(c_{k+1}-c_k)$

 		$r_k \leftarrow f^{-1}_i(I_k)$

​		$b_k \leftarrow f^{-1}_i(s[c_k])$

​	$b_N \leftarrow \infty$

​	**return** $[r_0:r_{N-1}],[b_0:b_N]$

**function** Quantize($w_n$, [$r_0:r_{N-1}$], [$b_0:b_N$])

​	**return** $r_k$ for k 满足条件 $b_k \le w_n < b_{k+1}$

* $N$：级数
* $N_w$：权重的数量
* $w_n$：第n个权重值
* $i_n$：第n个权重的重要性
* $f_i$：映射函数的重要性
* $c_i$：集群的边界索引
* $S$： 整体的加权熵

---

这里需要注意的是，算法1只对非负的权值进行量化。这是因为加权熵理论的局限性所导致，我们无法获得既有负面又有非负面的聚类结果。因此，我们将权重分为两类，一类为负权重，另一类为非负权重。并且，使用我们的算法进行量化时，每组都进行$\frac{N}{2}$级量化。

算法开始时，我们计算了每个权重的重要性(第2和3行)。这个过程有映射函数$f_i$完成，其使用$w_k$计算得到$i_k$。这个过程中，我们的经验使用的是平方函数$f_i(w) = w^2$来计算每个权重的重要性。在获得了所有权重的重要性值之后，我们会按增序的方式对结果进行排序(第4行)。

基于对重要性值的排序，算法会对集群进行边界索引的初始化从$c_0$到$c_N$(第5行)，

> 集群边界索引决定了权重属于哪个集群。准确的来说，集群$C_i$中包含了权重数组s中第$c_i$个权重到第$c_{i+1}-1$个权重(索引以0开始)。

因此：

1. 每个集群都具有相同数量的权重值

2. $C_{i+1}$中权重的重要性要高于$C_i$中的权重。


这个可以简单的将重要性排序后的数组s分为N份，并制定给每个集群。例如：$s=[1,2,3,4]$，N为2，我们可以设置$c_0=0, c_1 =2,c_2=4$，所以$C_0=$ {1, 2}，$C_1=${3, 4}。

在开始对进行集群边界初始化时，我们在新集群边缘迭代的执行增量搜索(第6行到第11行)。每次迭代中，每个集群$C_i$和其边界$c_i$ 和$c_{i+1}$，我们会使用二分法在$c_{i-1}$到 $c_{i+1}$ 间查找$c_i$。对每个集群的候选边缘索引为$c'_i$，我们都会重新计算集群$C_{i-1}$ 和$C_i$的加权熵(新边界值只对加权熵有影响)，当心的加权熵$S'$大于当前的加权熵，则将边界更新为$c'_i$。

在获取新集群边界之后，我们需要计算每个集群中$C_k$ 的重要性(第13行)。我们获得集群$C_k$中表示权重的值$r_k$(第14行)。为了界定那些权重属于哪些集群，我们可以通过集群边缘权值——$b_k$进行判别(以及进行权值量化)(第15行)；例如，集群$C_i$包含有w个权值，w需要满足条件$b_k \le w < b_{k+1}$。函数Quantize实现了量化过程。简而言之就是，给定一个权重$w_n$，其会为相关集群$c_k$产生表达权重值$r_k$。

基于加权熵的聚类可以提供满意的级数，该级数为在第3节所序。最大化加权熵，在考虑数据的重要性的同时优化量化结果(以使熵最大化)。因此，考虑到0附近的值重要性很低，我们的方法将0附近的值都分到一个大集群中。虽然该集群涵盖了大量的权重值，但只是将不常见的值分到一个集群中而已。




### 4.2 激活输出量化

激活输出的量化方式与权重的量化完全不同。因为权重在训练过程结束之后就已经固定，但是前向预测的激活输出与运行时给定的输入数据有很大的关系。这使得激活不太适合通过基于聚类的方法进行量化，因为聚类的方法需要稳定分布。

根据我们的调研，基于对数的量化(LogQuant)对于激活输出的量化有较好的结果。LogQuant也有利于最大限度地降低成本(例如：专用硬件加速器)，因为它可以将乘法转换成按位移位操作(例如：$w\times2^x=w\kern -.3em < \kern -.3em <x$)。不过，LogQuant方法并不提供某种有效的搜索策略，来查找网络中每个层中最优的LogQuant参数(比如：底数和偏移)。

---

**算法2：激活输出量化**

**function** BinaryToLogQuant($a_n$)

​	**return** $round(\frac{16\times log_2a_n-fsr}{step}) +1$

**function** LogQuantToBinary(index)

​	**if** index == 0 **then**

​		**return** 0

​	**else**

​		**return** $2^{\frac{1}{16}\times (fsr +step·(index-1))}$

**function** WeightedLogQuantReLU($a_n$)

​	**if** $a_n$ < 0 **then**

​		**return** 0

​	level_idx $\leftarrow BinaryToLogQuant(a_n)$

​	**if** level_idx $\le$ 0 **then**

​		**return** 0

​	**else if** level_idx $\ge$ N - 1 **then**

​		**return** LogQuantToBinary(N - 1)

​	**else**

​		**return** LogQuantToBinary(level_idx)

**function** ReprImportance(index)

​	**return** LogQuantToBinary(index)

**function** RelativeFrequency(index, a)

​	**for** k = 0 to$N_a$ - 1 **do**

​		level_idx$_k \leftarrow$ BinaryToLogQuant($a_n$)

​	**if** index == 0 **then**

​		**return** $\left| \{ a_n| level\_idx_n\le0\}\right|$

​	**else if** index == N - 1 **then**

​		**return** $\left|\{ a_n | level\_idx_n \ge N -1 \}  \right|$

​	**else**

​		**return** $\left|\{ a_n | level\_idx_n =index \}  \right|$

* $N:$ 级数
* $N_a$：激活输出的总量
* $a_n$：激活输出中第n个值
* $fsr$：最优的fsr值(整型)
* $step$：最优的step值(2的倍数)

---

我们对激活输出量化分为两步：修改过的 LogQuant方法和一个快速搜索LogQuant参数的策略。算法2展示我们在修改过的 LogQuant方法中使用到的主要函数。

首先，我们修改了原始LogQuant方法用来提高整体的精确度和稳定性。与传统的LogQuant方法不同，我们采用了更小的对数底($\frac{1}{8}$的倍数)和偏移($\frac{1}{16}$的倍数)，分别对应算法2中的‘step’和'fsr'。我们将第一量化级别定为零激活，并且其他的对数值对应相应的等级。例如，当我们要对激活输出进行3位量化时，第一级为0，第二级为$2^{\frac{fsr}{16}}$，第三级为$2^{\frac{fsr+step}{16}}$，以此类推。简单起见，我们将我们的输出激活量化整合为线性单元激活函数的一部分，在算法2中的WeightedLogQuantReLU进行描述。

其次，为我们的LogQuant变体提供了一种参数搜索的方法，其能找到最合适的底数和偏移，来保证输出质量损失最小化。我们的想法是利用的权重量化中加权熵最大化的概念。算法2中也有计算重要性I的函数(ReprImportance)和计算相对频率P的函数(RelativeFrequency)(这两个部分是用来计算加权熵到的)。训练过程中，为了在LogQuant中最大化给定每层激活输出的加权熵，我们对“fsr”和“step”进行仔细的搜索，因为可能的基数和偏移量通常都很小(例如，我们的经验是底数为16，偏移在500左右)。



### 4.3  将权重/激活量化集成入训练过程

我们将权重/激活输出量化的方法集成到传统神经网络的训练过程中。由于在每个小批量作为输入期间权重不变，所以可以通过每个小批量结束时更新之后的权重进行重量化。这里需要注意的是，我们在权重更新过程中使用全精度进行，之前的工作同样使用全精度。

另外，激活输出量化需要应用到前向/后向的传递过程中，因为每个传递过程都有一组属于自己的激活输出值。对于每一层，我们首先执行前向过程，然后使用普通的ReLU(不带LogQuant)。激活输出的结果会传入到我们的算法中，用于对LogQuant参数的搜索。通过算法找到最优的底数/偏移组合，然后使用WeightedLogQuantReLU函数对激活输出进行量化。激活输出的量化结果将传递到下一层中，网络中其他的层也会进行相同的操作。

在我们的训练框架中，任何网络都能通过我们的量化方式收益，而且不需要对网络进行修改。这使得将量化应用到网络上要简单的多，量化后的网络可以大大减少前向预测的所需的时间。现有方法不太实用，因为它们需要在设计时对网络进行修改，这就需要耗费大量的人力。



## 5. 实际操作

我们从神经网络的三个应用领域来评估我们的方案：图像分类，物体识别和语言建模。我们对Caffe进行了修改，从而使所有网络采用我们的方案；这里我们使用TensorFlow处理语言建模，并实现了LSTM框架。

> 修改后的Caffe代码可以在 https://github.com/EunhyeokPark/script_for_WQ 中看到

我们将精度损失约束在1%以内，旨在找到在满足精度约束的同时，给出最小位宽的量化配置。为了方便起见，我们使用(x, y)符号：x表示权重的位宽，y表示激活输出的位宽。这个表示法中`f`代表全精度。例如，(1, f)则表示权重使用1位表示，激活输出使用全精度表示。



### 5.1 图像分类: AlexNet, GoogLeNet和ResNet-50/101

对于图像分类任务来说，我们使用两个广为人知的网络(AlexNet和GoogLeNet)来测试我们的量化方案(都是用Caffe框架进行)，以及ResNet。

> ResNet的模型可以从https://github.com/KaimingHe/deep-residual-networks 处获取

为了在这几个网络使用我们的量化方案，我们在批次尺寸设置为256(AlexNet), 64 (GoogLeNet)和16 (ResNet-50/101)，结合微调的方式使用我们的方案进行量化。在GoogLeNet和ResNet的测试中，批次尺寸受到了GPU内存不足的影响；这可能会增加整体的精度损失。我们使用ILSVRC2012数据集，其包含128万张训练图片，和5万张测试图片。在微调的前6个周期，我们首先将初始化学习率设置为0.001，然后每两轮降低10倍。

下面的子章节中，我们会展示两种评测结果。首先，我们会证明我们的量化方式对于整个网络(全网络量化)是有用的，这在其他的方法中是无法做到的。其次，我们对除了第一层和最后一层使用我们的方案，与部分网络量化的方案进行比较，这里第一层和最后一层使用全精度表示。



#### 5.1.1 量化整个网络

图2用我们的方法对量化后的CNN的网络进行精度测试。可以看到对CNN进行限制位宽的量化可以获得较高的精度。

![](images/AlexNet.png)

![](images/GoogLeNet.png)

![](images/ResNet.png)

> 图2 Top-1和Top-5都是在微调模型后，进行量化的精度。虚线代表使用全精度算法网络的基线准确度。

AlexNet，最好的量化配置使用最窄的位宽在top-5上精度下降只在1%以内，这几种配置为(3, 6), (4, 4), (4, 5)和(4, 6)。例如，(4, 4)都减少了表示位宽，其精度为87.5%(= 1 - 4/32)，相较top-5下降了不到1%的精度。此外，与以前的工作相比，我们的方法提供了更低的等精度位宽。例如，LogQuant能达到75.1%，测试top-5使用4位表示权重，5位表示激活输出；Qiu等人使用8位表示权重和激活输出，在top-5上达到76.6%，top-1上达到53.0%。我们方法在使用2位表示权重，3位表示激活输出的情况下，就可达到相似的精度，top-1为51.37%，top-5为75.49%。

GoogLeNet，将精度损失严格的控制在1%内，我们的量化方式使用4-5位表示权重，6位表示激活输出。我们也观察到，在同等位宽限制下GoogLeNet的精度下降要比AlexNet大的多。我们认为这是因为GoogLeNet的模型规模比AlexNet更紧凑造成的，GoogLeNet的计算量要大于AlexNet，从而使得GoogLeNet权重的精度的下降很明显。即使这样，我的方案依旧比全精度实现的模型小5倍。

ResNet，据我们所知，本文是第一篇对具有50层和101层的网络进行整体量化进行记录的文章。两个网络即使在权重量化之后，也能保持相似的准确性，例如3位。不过，我们观察到，越深的网络就需要更多的位数来表示激活输出数据，比如6位，这可能是因为量化错误在较深的网络中会进行积累。



#### 5.1.2 量化部分网络

这节中，我们将我们的方法与目前最先进的两种方式(XNOR-Net和DoReFa-Net)进行了对比。为了公平起见，我们的比较方式是，网络中除了第一层和最后一层以外的层都进行量化。这里要注意的是，比较还并非公平。

1. 之前方式的最好结果中，权重使用1位表示，激活输出使用k位表示；而我们的方式是使用k位表示权重和激活输出。
2. XNOR-Net和DoReFa-Net都对网络进行了修改，而我的方法要包括ReLU层(在激活输出量化)。


图3比较了XNOR-Net和DoReFa-Net两种方式。

![](images/XNOR-Net-and-DoReFa-Net.PNG)

> 图3 这里使用对AlexNet网络进行量化，并对精度进行比较。权重量化(*Weighted Quantization*)表示我们的方案，这里‘X’和'D'分别表示 XNOR-Net和DoReFa-Net。虚线表示使用全精度网络得到的精度基线。

图3展示了 XNOR-Net和DoReFa-Net方案使用二值权重的比较，例如(1, f)，1位的权重损失也非常小，其仅限于二值量化和全精度激活输出，在严格的精度损失约束下，不能很好的对精度和性能进行权衡。XNOR-Net 二值化权重和激活输出的量化，例如(1, 1)，这样过低的降低了准确性，而我们的方法提供了满足精度需求的多位量化。 DoReFa-Net通过对激活输出的多位量化缓解了这样的限制。不过，在相似的配置下，我们的方式(2位表示权重，3位表示激活输出)在top-1方面要优于DoReFa-Net(1位表示权重，4位表示激活输出)0.69个百分点(0.69%)。总之，我们的方式使用了更加灵活的量化配置，可以使用比之前算法更窄的位宽，这对于需要在严格精度约束下进行高效推理的系统非常有用。



#### 5.1.3 压缩分析

|                            | Weights | Weights | Weights | Activations | Top-1 |
| :------------------------: | :-----: | :-----: | :-----: | :---------: | :---: |
|                            |  P [%]  |  Q[MB]  |   +H    |   Q [MB]    |  [%]  |
|          WQ(4,4)           |    -    |  30.5   |  18.1   |    0.47     | 55.8  |
|          WQ(2,3)           |    -    |  15.3   |  12.5   |    0.35     | 53.7  |
|          XNOR-Net          |    -    |  23.7   |    -    |    0.72     | 44.2  |
|         DoReFa-Net         |    -    |  23.6   |    -    |    0.47     | 53.0  |
|      Deep Compression      |   11    |   8.9   |   6.9   |    3.75     | 57.2  |
| Deep Compression + WQ(4,6) |   11    |   8.3   |   6.5   |    0.70     | 56.3  |

> 表1 压缩AlexNet所需的内存(P：裁剪比例，Q：量化，H：哈夫曼编码)。

表1比较了我们的方法和现有的方法。从表格中我们观察到如下现象

首先，XNOR-Net和DoReFa-Net要比我们的方法(WQ)有更多的权重，这是因为他们没有量化第一层和最后一层。另外，当使用全精度和二值化权重对于哈夫曼编码并没有什么帮助。

其次，当WQ以用于裁剪时，激活和权重(分别为8.9MB和8.3MB)会缩小5.4倍，另外进度损失为0.9%。我们在激活输出部分获得了比权重更多的位宽缩小，因为利用深度压缩利用的是全精度激活输出，而我们使用的是6位激活输出。



#### 5.1.4 分层量化的可行性研究

在之前的章节我，我们对网络中所有层使用相同的位宽表示。不过，根据我们的观察，不同的层对于量化位宽的敏感度不同。因此，我们研究层级(*layer-wise*)量化的可行性，也就是不同的层使用不同的位宽表示。在这项研究中，我们基于AlexNet评估了四种为每层指定位宽的方式：单调递减(Dec)，单调递增(Inc)，凹形(Doncave)和凸形(Convex)。所有四种方案都被设计成都具有相同的位数。例如，单调递减方法对第一个卷积层使用6位表示权重/激活输出，那么其他方法也一样；同时，对最后一个全连接层使用2位表示权重/激活输出，那么其他方法也一样。

|          |  Dec  |  Inc  | Concave | Convex |
| :------: | :---: | :---: | :-----: | :----: |
| Top-1[%] | 53.79 | 50.35 |  54.45  | 54.33  |
| Top-5[%] | 77.59 | 74.89 |  76.43  | 78.20  |

> 表2 在不同的层级量化方式下，我们的方法的精度比较。

如表2所示，我们观察到使用较少位来表中间的层(Convex)能获得较高的精度，但是给输入附近的层给予更少的位宽(Inc)精度则会有很大的损失。Zhou等人也观察到了相同的情况。我们相信，在量化过程中考虑这种层级的敏感度，从而考虑更大的位宽进行优化的方案是可行的。对所有可能的位宽组合进行逐一测试不太现实，因为即使是小型网络，也会有很多组合(例如：AlexNet每层从2位到6位的量化具有$5^{15} \approx 3 \times 10^{10}$种配置)。快速探索分层量化的算法留在未来进行。



### 5.2 物体检测：基于*ResNet-50*的*R-FCN* 

为了评估我们的量化方法对更复杂的视觉任务的有效性，我们使用了具有50层的R-FCN模型进行物体检测。R-FCN模型结合了一种残差网络(ResNet)和Faster R-CNN网络(使用区域匡定位的方式进行物体分类)。在全精度模型上，我们用量化方法进行微调。

因为量化误差是在深层积累的，所以深层模型是难以量化的，但是我们的方法成功量化了50层的物体检测模型，并且精度损失非常小。

![](images/Figure_4.PNG)

> 图4 R-FCNd的mAP结果。虚线为基准线，是使用全精度网络计算出的精度。

图4中(5, 6)的配置对于[mAP](https://www.zhihu.com/question/53405779)的损失只有0.51%，同时将模型的体积和计算量减少了5倍以上。我们也观察到，使用我们的方法时，激活输出通常会需要比权重更高的位数表示(比如：使用6位来表示激活输出，4或5位表示权重也可以达到比较稳定和满意的效果)。我们认为这是因为R-FCN中使用了简单的边界框回归机制(直接从指定网络区域中直接提取边框)，因此才会对激活输出的精度很敏感。我们会在以后来对该情况进行调研。

我们未来的工作中，我们将会研究我们的方法在更深的网络上的可行性。根据我们对基于ResNet-100的R-FCN网络的初步研究，当用PASCAL VOC数据集对模型进行微调时，我们未能在损失合理的精度的前提下获得量化级数。我们认为，这个是由于转移学习和量化在一个非常深的网络上的混合效应，这需要在深度网络上对量化进行进一步的研究。



### 5.3 语言建模：一种LSTM网络

为了验证我们的方法是否能用于递归神经网络，将方法应用到用于语言建模的LSTM网络进行初步分析，这里我们使用TensorFlow进行验证。我们评估了三种尺寸的RNN网络，小型尺寸(200个隐藏单元和20个时间步数)，中型尺寸(650个隐藏单元和35个时间步数)，大型网络(1500个隐藏单元和35个时间步数)，这些网络都只有两层。我们使用Penn Tree Bank数据集对这个三个RNN网络进行量化之前/之后进行测试，记录网络的词级困惑度。

|       | Large | Large | Medium | Medium | Small  | Small  |
| :---: | :---: | :---: | :----: | :----: | :----: | :----: |
|       | Valid | Test  | Valid  |  Test  | Valid  |  Test  |
| float | 82.77 | 78.63 | 87.69  | 83.54  | 119.19 | 114.46 |
| 1-bit | 92.20 | 88.48 | 104.0  | 100.7  | 147.19 | 141.07 |
| 2-bit | 86.73 | 82.90 | 92.49  | 89.24  | 137.34 | 131.15 |
| 3-bit | 85.59 | 81.57 | 86.73  | 83.50  | 121.21 | 117.00 |
| 4-bit | 81.83 | 78.09 | 88.01  | 83.84  | 121.84 | 114.95 |

> 表3 对一个语言建模LSTM模型进行量化后，记录词级困惑度(概率越高, 困惑度越低, 语言模型越好)。

表3展示了LSTM网络使用全精度(float)和量化后的词级困惑度。结果展示了4位量化的权重与全精度的实现结果非常相近。同样，我们方法提供了选项，可以使用更窄的位宽，进一步的减小模型尺寸和计算量，不过代价就是要降低输出的质量(例如：更加高的困惑度)。



## 6. 总结

本文中，我们提供了一种基于加权熵实的权重/激活输出量化实现。我们的方法有两大好处:

1. 灵活的多位量化，允许我们在很小的精度下降的前提下对神经网络进行优化。
2. 自动量化，不需要对网络进行修改。

更具我们很多实际神经网络评估结果(AlexNet, GoogLeNet, ResNet-50/101, R-FCN和某个LSTM网络)，我们方法能让保证精度下降在1%内(top-5或mAP)。使用4位权重/激活输出表示AlexNet，4/5位权重和6位激活输出(GoogLeNet, ResNet和R-FCN)，都能保证精度下降在1%。下一步我们的调研将会是特别深的神经网络模型(例如：ResNet-152)，以及对RNN模型的量化。



## 鸣谢

该研究得到韩国国家研究基金会(NRF-2016R1A2B3009361)和三星电子(三星尖端技术研究所(*SAIT*)和三星研究资助中心 SRFC-TC1603-04)资助。

