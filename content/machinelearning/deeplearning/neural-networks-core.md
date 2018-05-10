---
title: "neural-networks-core"
date: 2018-02-21 22:05
---
[TOC]
# 我的证书
 - [Neural Networks and Deep Learning](https://www.coursera.org/account/accomplishments/records/HZ7JCCSRTUVR)
 - [Improving Deep Neural Networks: Hyperparameter tuning, Regularization and Optimization](https://www.coursera.org/account/accomplishments/records/RGFAH9M7HG99)
 - [Sequence Model](https://www.coursera.org/account/accomplishments/records/NVWZ2BPTVP9T)
# 我的简略神经网络教程
 - [神经网络](/static/pdf/Neural-networks-and-deep-learning.pdf)
 - [源代码](https://github.com/kitianFresh/neural-networks-by-c)

# 申明 
本打算自己总结笔记，但是课上完之后太久没复习，发现这位同学已经总结很好了，所以大部分都借鉴了这个[笔记](http://kyonhuang.top/Andrew-Ng-Deep-Learning-notes/#/Improving_Deep_Neural_Networks/%E4%BC%98%E5%8C%96%E7%AE%97%E6%B3%95)，稍微加了自己的理解, 自己偷懒了一波。

# Train/Dev/Test
## 数据集划分
数据样本划分一下几部分：

 - 训练集(Train set):　用来训练模型的数据集合
 - 验证集(Development set): 利用验证集（有时候叫cross validation set） 进行交叉验证，选出最好的模型。一般会用来进行交叉验证来选取出组好的超参数。
 - 测试集(Test set): 在训练完成之后对测试集合进行测试，获取模型运行的无偏估计。该集合是用来评估我们的模型好坏的。

在小数据集和传统机器学习时代，比如100,1000,10000,100000等，数据集划分比例一般是:

 - 无验证集合情况：　70%/30%
 - 有验证集合情况：　60%/20%/20%
在大数据时代以及深度学习时代，数据集合规模达到百万级别，验证集合和测试集合比重趋于更小。

验证集的目的是评估不同的模型(一般是不同的超参数得到的模型)那种更加有效，所以验证集合只要足够大到可以验证2-10种模型那种更好，不需要使用20%数据集验证，百万数据集抽取１万即可验证。

测试集的目标是评估模型效果，所以当数据集百万的时候也只需要１%就足够了。以上的少量不能随意，要剧本和训练集一样的多样性和相同的分布，虽然只有１万个，但是要具备多样性，要能够代表你数据集合整体。

 - 100万数据:  98%/1%/1%
 - 百万以上：　99.5%/0.25%/0.25%

## 注意
**验证集要和训练集来自于同一个分布**（数据来源一致），可以使得机器学习算法变得更快并获得更好的效果。
如果不需要用无偏估计来评估模型的性能，则可以不需要测试集

# 偏差bias 和 方差 variances
## 偏差-方差分解
“偏差-方差分解”（bias-variance decomposition）是解释学习算法泛化性能的一种重要工具。

泛化误差可分解为偏差、方差与噪声之和:

 - 偏差：度量了学习算法的期望预测与真实结果的偏离程度，即刻画了**学习算法本身的拟合能力**；
 - 方差：度量了同样大小的训练集的变动所导致的学习性能的变化，即刻画了**数据扰动所造成的影响**；
 - 噪声：表达了在当前任务上任何学习算法所能够达到的期望泛化误差的下界，即刻画了**学习问题本身的难度**。
偏差-方差分解说明，**泛化性能是由学习算法的能力、数据的充分性以及学习任务本身的难度所共同决定的**。给定学习任务，为了取得好的泛化性能，则需要使偏差较小，即能够充分拟合数据，并且使方差较小，即使得数据扰动产生的影响小。


## 过拟合与欠拟合
 - 过拟合: 训练集上的误差非常小，测试集上的误差非常大
 - 欠拟合: 训练集和测试集上的误差都比较大且两者相当
在欠拟合（underfitting）的情况下，出现高偏差（high bias）的情况，即不能很好地对数据进行分类。

当模型设置的太复杂时，训练集中的一些噪声没有被排除，使得模型出现过拟合（overfitting）的情况，在验证集上出现高方差（high variance）的现象。

当训练出一个模型以后，如果：

 - 训练集的错误率较小，而验证集的错误率却较大，说明模型存在较大方差，可能出现了过拟合；
 - 训练集和开发集的错误率都较大，且两者相当，说明模型存在较大偏差，可能出现了欠拟合；
 - 训练集错误率较大，且开发集的错误率远较训练集大，说明方差和偏差都较大，模型很差；
 - 训练集和开发集的错误率都较小，且两者的相差也较小，说明方差和偏差都较小，这个模型效果比较好。

需要指出的是，误差的概念是相对的，不是说15% 误差就一定是不好，他是相对于我们实际最优误差而言的，如果你的误差比这个可接受的最优误差好或者接近，则也是好的。最优误差通常也称为“贝叶斯误差”。

# 解决方案
## 高方差(过拟合)
### 正则化 Regularization
正则化在机器学习中非常适用，通过对参数引入惩罚，减低模型复杂度，提高模型泛华能力，解决高偏差问题。
逻辑斯特回归的正则化例子：

$$
J(w,b) = \frac{1}{m}\sum_{i=1}^mL(\hat{y}^{(i)},y^{(i)})+\frac{\lambda}{2m}{||w||}^2_2
$$

 - L2 正则化
$$
\frac{\lambda}{2m}{||w||}^2_2 = \frac{\lambda}{2m}\sum_{j=1}^{n_x}w^2_j = \frac{\lambda}{2m}w^Tw
$$

 - L1 正则化
$$
\frac{\lambda}{2m}{||w||}\_{1} = \frac{\lambda}{2m}\sum_{j=1}^{n_x}{|w_j|}
$$

神经网络正则化例子：
成本函数：
$$
J(w^{[1]}, b^{[1]}, ..., w^{[L]}, b^{[L]}) = \frac{1}{m}\sum_{i=1}^mL(\hat{y}^{(i)},y^{(i)})+\frac{\lambda}{2m}\sum_{l=1}^L{{||w^{[l]}||}}^2_F
$$

其中　$ w^{[l]} $ 的形状为 $ (n^{[l-1]}, n^{[l]}) $. 有如下公式：
$$
{{||w^{[l]}||}}^2\_{F} = \sum^{n^{[l-1]}}\_{i=1}\sum^{n^{[l]}}\_{j=1}(w^{[l]}\_{ij})^2
$$

正则化可以让参数和损失一起减小，这样可以让某些参数不会太大，就可以防止过多参数对某些特征维度的过度关心和学习，减小对噪声的学习，从而提高泛化能力。

### 反向随机失活 Inverted Dropout
Dropout 是神经网络模型的另外一种正则化手段，一般用在CNN。就是在训练过程中随机去除某些节点，不参与权重更新。这个效果会使得我们的神经元不会过度依赖某些输入数据特征，也就不会给某些权重设置的过大，达到和L2正则一样的权重收缩效果。
实现dropout的代码是反向随机失活，这里有一个技巧是对输出的激活值除以保留概率:
```python
keep_prob = 0.8    # 设置神经元保留概率
dl = np.random.rand(al.shape[0], al.shape[1]) < keep_prob
al = np.multiply(al, dl)
al /= keep_prob
```
最后一步`al /= keep_prob`是因为 $a^{[l]}$ 中的一部分元素失活（相当于被归零），为了在下一层计算时不影响 $Z^{[l+1]} = W^{[l+1]}a^{[l]} + b^{[l+1]}$ 的期望值，因此除以一个`keep_prob`。

注意，**在测试阶段不使用 dropout**，因为那样会使得预测结果变得随机。

### 数据集扩增 Data Augmentation
通过图片的一些变换（旋转，平移，对称，扭曲，翻转，局部放大后切割等），得到更多的训练集和验证集。
### 提前终止训练 Early Stop
将训练集和验证集进行梯度下降时的成本变化曲线画在同一个坐标轴内，在两者开始发生较大偏差时及时停止迭代，避免过拟合。这种方法的缺点是无法同时达成偏差和方差的最优。

## 高偏差(欠拟合)
 - 增大数据集
 - 减少正则数量

## 数据归一化(Normalizing inputs)
归一化一般采用的是　`Z-score standardization` 即0均值标准化。
$$
x = \frac{x - \mu}{\sigma}
$$
其中，
$$
\mu = \frac{1}{m}\sum^m_{i=1}x^{(i)}
\sigma = \sqrt{\frac{1}{m}\sum^m_{i=1}x^{{(i)}^2}}
$$
由于模型的形状不一样，在不使用标准化的成本函数中，如果设置一个较小的学习率，可能需要很多次迭代才能到达全局最优解；

而如果使用了标准化，那么无论从哪个位置开始迭代，都能以相对较少的迭代次数找到全局最优解。
<div align="center"><img src="/static/images/ML/DL/nn-cores/why_normalize.png" style="width:700px;height:500px;">
<caption><center> 归一化前后数据空间变化 </center></caption></div>

## 梯度消失与爆炸(Vanishing/Exploding Gradients)
在梯度函数上出现的以指数级递增或者递减的情况分别称为梯度爆炸或者梯度消失。
假设　$ g(z) = z, b^{[l]} = 0 $, 对目标输出：
$$
\hat{y} = W^{[L]}W^{[L-1]}...W^{[2]}W^{[1]}X
$$
 - 对于 $ W^{[l]} $的值大于 1 的情况，激活函数的值将以指数级递增；
 - 对于 $ W^{[l]} $的值小于 1 的情况，激活函数的值将以指数级递减。

## 权重初始化(Weights Initialization)
我们在每一层输出：
$$
z={w}_1{x}_1+{w}_2{x}_2 + ... + {w}_n{x}_n + b
$$
希望ｚ尽量较小。当输入数量n较大的时候，每个 $w_i$ 较小的话，就可以避免ｚ过大。两种初始化权重方法:
### Xavier Initialization
```python
WL = np.random.randn(WL.shape[0], WL.shape[1]) * np.sqrt(1/n)
```
采用激活函数一般是 `tanh`. 设置　`Var(wi)=1/n`　就是　`Xavier Initialization`
### He Initialization
采用激活函数一般是 `Relu`, 设置　`Var(wi)=2/n` 就是　`He Initialization`
## 梯度检查(Gradient Checking)
梯度检查采用双侧差值法，因为更精确，单侧差值法不够精确。
$$
d\theta_{approx}[i] ＝ \frac{J(\theta_1, \theta_2, ..., \theta_i+\varepsilon, ...) - J(\theta_1, \theta_2, ..., \theta_i-\varepsilon, ...)}{2\varepsilon}
$$

$$
\approx{d\theta[i]} = \frac{\partial J}{\partial \theta_i}
$$

梯度检查:
$$
\frac{{||d\theta\_{approx} - d\theta||}\_2}{{||d\theta\_{approx}||}\_2+{||d\theta||}\_2}
$$

梯度检查注意事项:

 - 不要在训练中使用梯度检验，它只用于调试（debug）。使用完毕关闭梯度检验的功能；
 - 如果算法的梯度检验失败，要检查所有项，并试着找出 bug，即确定哪个 dθapprox[i] 与 dθ 的值相差比较大；
 - 当成本函数包含正则项时，也需要带上正则项进行检验；
 - 梯度检验不能与 dropout 同时使用。因为每次迭代过程中，dropout 会随机消除隐藏层单元的不同子集，难以计算 dropout 在梯度下降上的成本函数 J。建议关闭 dropout，用梯度检验进行双重检查，确定在没有 dropout 的情况下算法正确，然后打开 dropout；

# 神经网络优化(Neural Network Optimization)
目前的神经网络模型都需要庞大的数据集训练才会有惊人的效果，才能秒杀传统机器学习。但是大数据集合训练就很化时间，这里我们提出了很多神经网络加速和优化算法，提高模型研发效率.
## BGD/SGD/mini-batch-GD
这种方法来自传统机器学习方法，例如逻辑斯特回归模型也通常会采用SGD 或者　mini-batch-GD 进行加速训练。

batch 梯度下降BGD是我们计算损失最原始的那个公式，是要一次计算所有样本损失的和，完成一次迭代训练要用完所有的样本。因此当数据集很大的时候，比如百万，计算就非常耗时了。他是一种利用全局信息更新参数的方法，成本下降不会出现频繁抖动，但是会非常耗时。

因此饿哦们提出了　mini-batch-GD 的方法，更新一次参数不使用全部数据，而是随机采样一小批数据进行，这样就可以加快迭代速度了。
<div align="center"><img src="/static/images/ML/DL/nn-cores/training-with-mini-batch-gradient-descent.png" style="width:700px;height:500px;">
<caption><center> training-with-mini-batch-gradient-descent </center></caption></div>

甚至还有随机梯度下降SGD，这个时候我们使用一个样本就更新一次参数，这种方式支持在线更新模型，而且训练速度快，但是容易发生抖动，而且可能跳过最优解。此时就相当于mini-batch 大小取１.
### batch 大小影响
 - mini-batch 的大小为 1，即是随机梯度下降法（stochastic gradient descent），每个样本都是独立的 mini-batch；
 - mini-batch 的大小为 m（数据集大小），即是 batch 梯度下降法；

<div align="center"><img src="/static/images/ML/DL/nn-cores/choosing-mini-batch-size.png" style="width:700px;height:500px;">
<caption><center> choosing-mini-batch-size </center></caption></div>

batch 梯度下降法：

 - 对所有 m 个训练样本执行一次梯度下降，每一次迭代时间较长，训练过程慢；
 - 相对噪声低一些，幅度也大一些；
 - 成本函数总是向减小的方向下降。

随机梯度下降法：

 - 对每一个训练样本执行一次梯度下降，训练速度快，但丢失了向量化带来的计算加速；
 - 有很多噪声，减小学习率可以适当；
 - 成本函数总体趋势向全局最小值靠近，但永远不会收敛，而是一直在最小值附近波动。

### mini-batch 大小的选择
 - 如果训练样本的大小比较小，如 m ⩽ 2000 时，选择 batch 梯度下降法；
 - 如果训练样本的大小比较大，选择 Mini-Batch 梯度下降法。为了和计算机的信息存储方式相适应，代码在 mini-batch 大小为 2 的幂次时运行要快一些。典型的大小为 26、27、...、29；
 - mini-batch 的大小**要符合 CPU/GPU 内存**。

### mini-batch 的通用步骤
 1. 混洗shuffle数据集
 2. 分割数据集为一个个mini-batch
```python
m = X.shape[1] 
permutation = list(np.random.permutation(m))
shuffled_X = X[:, permutation]
shuffled_Y = Y[:, permutation].reshape((1,m))
```

## 指数加权移动平均与校正(Exponetially weighted averages and bias correction)
这个算法是统计学里面的一个经典算法，就是利用过去的累计数据来估算当前的数据，这个算法认为当前变量的值是由过去的值和现在的值共同决定的，以此来拟合真实数据；但是这个不是用来做预测的，他只是根据历史数据得到一条拟合曲线。这个曲线用来反应某种时间序列的变化趋势。

$$
v_t = 
\begin{cases} 
0, &t = 0 \\\\
\beta v_{t-1} + (1-\beta)\theta_t, &t > 0 
\end{cases}
$$

给定一个时间序列，例如伦敦一年每天的气温值，图中蓝色的点代表真实数据。对于一个即时的气温值，取权重值 $\beta$ 为 0.9，根据求得的值可以得到图中的红色曲线，它反映了气温变化的大致趋势。

当取权重值 $\beta=0.98$ 时，可以得到图中更为平滑的绿色曲线。而当取权重值 $\beta=0.5$ 时，得到图中噪点更多的黄色曲线。**$\beta$ 越大相当于求取平均利用的天数越多，曲线自然就会越平滑而且越滞后; $\beta$　越小相当于求品均利用的天数越少，曲线会抖动厉害，但是更快适应温度变化**。

<div align="center"><img src="/static/images/ML/DL/nn-cores/Exponentially-weight-average.png" style="width:700px;height:500px;">
<caption><center> Exponentially-weight-average </center></caption></div>

### 理解指数加权平均
我们将上述公式展开成关于 $\theta$ 的式子, 取 $\beta=0.9$：
$$
v_{100} = 0.1 \theta_{100} + 0.1 * 0.9 \theta_{99} + 0.1 * {(0.9)}^2 \theta_{98} + ...
$$
**指数平均加权并不是最精准的计算平均数的方法**，你可以直接计算过去 10 天或 50 天的平均值来得到更好的估计，但缺点是保存数据需要占用更多内存，执行更加复杂，计算成本更加高昂。

代码中 $v$ 的更新过程只需要参考前一个　$v$, 这样**非常节省内存，你不需要把所有的历史数据存储在内存中来计算平均值。**
$$
v := \beta v + (1 - \beta)\theta_t
$$

### 指数加权平均偏差修正
$$
v_0 = 0 \\\\
v_1 = 0.98v_0 + 0.02\theta_1
$$

因此，v1 仅为第一个数据的 0.02（或者说 1-β），显然不准确。往后递推同理。

因此，我们修改公式为:
$$
v_t = \frac{\beta v_{t-1} + (1 - \beta)\theta_t}{{1-\beta^t}}
$$

随着 t 的增大，$\beta$ 的 t 次方趋近于 0。因此当 t 很大的时候，偏差修正几乎没有作用，但是在前期学习可以帮助更好的预测数据。在实际过程中，一般会忽略前期偏差的影响。

## 动量梯度下降法(gradient descent with momentum)
**动量梯度下降（Gradient Descent with Momentum）**是计算**梯度的指数加权平均数，并利用该值来更新参数值**。具体过程为：

for l = 1, .. , L：
$$
v_{dW^{[l]}} = \beta v_{dW^{[l]}} + (1 - \beta) dW^{[l]} \\\\
v_{db^{[l]}} = \beta v_{db^{[l]}} + (1 - \beta) db^{[l]} \\\\
W^{[l]} := W^{[l]} - \alpha v_{dW^{[l]}} \\\\
b^{[l]} := b^{[l]} - \alpha v_{db^{[l]}}
$$

<div align="center"><img src="/static/images/ML/DL/nn-cores/Gradient-Descent-with-Momentum.png" style="width:700px;height:500px;">
<caption><center> Gradient-Descent-with-Momentum </center></caption></div>
蓝色曲线是普通梯度下降得到的曲线，由于存在上下波动，从而减缓下降速度，因此只能使用一个较小的学习率进行迭代。如果设置较大的学习率，结果可能会是紫色曲线偏离轨道。

而使用动量梯度下降时，通过累加过去的梯度值来减少抵达最小值路径上的波动，加速了收敛，因此在横轴方向下降得更快，从而得到图中红色的曲线。

当前后梯度方向一致时，动量梯度下降能够加速学习；而前后梯度方向不一致时，动量梯度下降能够抑制震荡。
### 为什么动量梯度得以奏效
利用了梯度的历史信息，得到梯度的走势。这些信息被形象的比喻为速度和加速度；这里的　$v_{dW}$ $v_{db}$ 就是速度， $dW$ $db$ 就是加速度；

速度通过过去的加速度信息不断的累加起来，得到过去的运行方向和走势，使得越陡峭的地方运行越快。

## RMSprop(均方根传播)
RMSProp (Root Mean Square Prop, 均方根支) 算法在对梯度进行指数加权平均基础上，又引入平方和平方根。具体公式如下，原来的梯度被平方，然后更新的时候又开根号:
$$
s_{dw} = \beta s_{dw} + (1 - \beta)(dw)^2 \\\\
s_{db} = \beta s_{db} + (1 - \beta)(db)^2 \\\\
w := w - \alpha \frac{dw}{\sqrt{s_{dw} + \epsilon}} \\\\
b := b - \alpha \frac{db}{\sqrt{s_{db} + \epsilon}} 
$$

其中，ϵ 是一个实际操作时加上的较小数（例如10^-8），为了防止分母太小而导致的数值不稳定。这种方式是机器学习中的一个常见技巧，我们在很多地方都见过了。

比如朴素贝叶斯算法，为了防止某些概率为０,我们为分子和分母引入一个常数因子，同时保证概率和还是１，但是保证了没有概率为０，即使很小。再比如　Softmax 为了防止不稳定和溢出，我们会先减去一个常数因子(一般是最大值)。在计算 tf-idf 的信息熵的时候，

当 dw 或 db 较大时，$(dw)^2$ $(db)^2$会较大，进而 $s\_{dw}$ $s\_{db}$ 也会较大，最终使得
$$
\frac{dw}{\sqrt{s_{dw} + \epsilon}}
$$
和
$$
\frac{db}{\sqrt{s_{db} + \epsilon}}
$$
较小，从而减小某些维度梯度更新波动较大的情况，使下降速度变得更快。
<div align="center"><img src="/static/images/ML/DL/nn-cores/RMSProp.png" style="width:700px;height:500px;">
<caption><center> RMSProp </center></caption></div>

RMSProp 有助于减少抵达最小值路径上的摆动，并允许使用一个更大的学习率 $\alpha$，从而加快算法学习速度。并且，它和 Adam 优化算法已被证明适用于不同的深度学习网络结构。


## Adam(Adaptive momentum estimation) = Momentum + RMSprop
**Adam 优化算法（Adaptive Moment Estimation，自适应矩估计）**基本上就是将 Momentum 和 RMSProp 算法结合在一起，通常有超越二者单独时的效果。最常用的。

$$
v_{dW} = 0, s_{dW} = 0, v_{db} = 0, s_{db} = 0
$$

用每一个 mini-batch 计算 dW、db，第 t 次迭代时：
$$
v_{dW} = \beta_1 v_{dW} + (1 - \beta_1) dW \\\\
v_{db} = \beta_1 v_{db} + (1 - \beta_1) db \\\\
s_{dW} = \beta_2 s_{dW} + (1 - \beta_2) {(dW)}^2 \\\\
s_{db} = \beta_2 s_{db} + (1 - \beta_2) {(db)}^2
$$

一般使用 Adam 算法时需要计算偏差修正:
$$
v^{corrected}\_{dW} = \frac{v_{dW}}{1-{\beta_1}^t} \\\\
v^{corrected}\_{db} = \frac{v_{db}}{1-{\beta_1}^t} \\\\
s^{corrected}\_{dW} = \frac{s_{dW}}{1-{\beta_2}^t} \\\\
s^{corrected}\_{db} = \frac{s_{db}}{1-{\beta_2}^t}
$$

W、b 的更新:
$$
W := W - \alpha \frac{v^{corrected}\_{dW}}{{\sqrt{s^{corrected}\_{dW}} + \epsilon}} \\\\
b := b - \alpha \frac{v^{corrected}\_{db}}{{\sqrt{s^{corrected}\_{db}} + \epsilon}}
$$

### 超参数的选择
Adam 优化算法有很多的超参数，其中

 - 学习率 α：需要尝试一系列的值，来寻找比较合适的；
 - β1：常用的缺省值为 0.9；
 - β2：Adam 算法的作者建议为 0.999；
 - ϵ：不重要，不会影响算法表现，Adam 算法的作者建议为 10−8；

β1、β2、ϵ 通常不需要调试。

## 学习率衰减(Learning rate decay)
如果设置一个固定的学习率 α，在最小值点附近，由于不同的 batch 中存在一定的噪声，因此不会精确收敛，而是始终在最小值周围一个较大的范围内波动。

而如果随着时间慢慢减少学习率 α 的大小，在初期 α 较大时，下降的步长较大，能以较快的速度进行梯度下降；而后期逐步减小 α 的值，即减小步长，有助于算法的收敛，更容易接近最优解。

最常用的学习率衰减方法：
$$
\alpha = \frac{1}{1 + decay\\_rate * epoch\\_num} * \alpha_0
$$
其中，`decay_rate`为衰减率（超参数），`epoch_num`为将所有的训练样本完整过一遍的次数。

指数衰减:
$$
\alpha = 0.95^{epoch\\_num} * \alpha_0
$$

其他衰减:
$$
\alpha = \frac{k}{\sqrt{epoch\\_num}} * \alpha_0
$$

# 超参数调试、Batch Normalizing
## 超参数调试处理
### 重要程度排序
目前已经讲到过的超参数中，重要程度依次是（仅供参考）：

最重要：

 - 学习率 α；

其次重要：

 - β：动量衰减参数，常设置为 0.9；
 - hidden units：各隐藏层神经元个数；
 - mini-batch 的大小；

再次重要：

 - β1，β2，ϵ：Adam 优化算法的超参数，常设为 0.9、0.999、10−8；
 - layers：神经网络层数;
 - decay_rate：学习衰减率；

### 调参技巧

- 随机选择点（而非均匀选取），用这些点实验超参数的效果。这样做的原因是我们提前很难知道超参数的重要程度，可以通过选择更多值来进行更多实验；
<div align="center"><img src="/static/images/ML/DL/nn-cores/Hyperparameter-debug.png" style="width:700px;height:500px;">
<caption><center> Hyperparameter-debug </center></caption></div>

- 由粗糙到精细：聚焦效果不错的点组成的小区域，在其中更密集地取值，以此类推；
<div align="center"><img src="/static/images/ML/DL/nn-cores/Hyperparameter-debug-coarse-to-fine.png" style="width:700px;height:500px;">
<caption><center> Hyperparameter-debug-coarse-to-fine </center></caption></div>

### 选择合适的范围
采用对数尺方法，我们的参数范围是 $10^a - 10^b$ 的话，我们可以在 [a, b] 区间产生随机数，然后带回原来的指数值。这样可以避免直接在原来区间取值的不均衡性。
```Python
r=a + (b-a)*np.random.rand()
```

 - 对于学习率 $\alpha$，用对数标尺而非线性轴更加合理：0.0001、0.001、0.01、0.1 等，然后在这些刻度之间再随机均匀取值；
 - 对于 $\beta$，取 0.9 就相当于在 10 个值中计算平均值，而取 0.999 就相当于在 1000 个值中计算平均值。可以考虑给 $1-\beta$ 取值，这样就和取学习率类似了。

上述操作的原因是当 β 接近 1 时，即使 β 只有微小的改变，所得结果的灵敏度会有较大的变化。例如，β 从 0.9 增加到 0.9005 对结果（1/(1-β)）几乎没有影响，而 β 从 0.999 到 0.9995 对结果的影响巨大（从 1000 个值中计算平均值变为 2000 个值中计算平均值）。

## Batch Normalizing
**批标准化（Batch Normalization，经常简称为 BN）**会使参数搜索问题变得很容易，使神经网络对超参数的选择更加稳定，超参数的范围会更庞大，工作效果也很好，也会使训练更容易。

我们对输入特征 X 使用了标准化处理。我们也可以用同样的思路处理**隐藏层**的激活值 $a^{[l]}$, 以加速　$W^{[l+1]}$ 和　$b^{[l+1]}$ 的训练。在实践中，我们经常选择标准化 $Z^{l}$：
$$
\mu = \frac{1}{m} \sum_i z^{(i)} \\\\
\sigma^2 = \frac{1}{m} \sum_i {(z_i - \mu)}^2 \\\\
z_{norm}^{(i)} = \frac{z^{(i)} - \mu}{\sqrt{\sigma^2 + \epsilon}}
$$

其中，m 是单个 mini-batch 所包含的样本个数，ϵ 是为了防止分母为零，通常取 10−8。

这样，我们使得所有的输入 $z^{(i)}$ 均值为 0，方差为 1。但我们不想让隐藏层单元总是含有平均值 0 和方差 1，也许隐藏层单元有了不同的分布会更有意义。因此，我们计算
$$
\tilde z^{(i)} = \gamma z^{(i)}_{norm} + \beta
$$

通过对 $\gamma$ 和 $\beta$ 的合理设置，可以让 $\tilde z^{(i)}$ 的均值和方差为任意值。这样，我们对隐藏层的 $z^{(i)}$ 进行标准化处理，用得到的 $\tilde z^{(i)}$ 替代 $z^{(i)}$。

**设置  $\gamma$ 和 $\beta$ 的原因是，如果各隐藏层的输入均值在靠近 0 的区域，即处于激活函数的线性区域，不利于训练非线性神经网络，从而得到效果较差的模型**。因此，需要用 γ 和 β 对标准化后的结果做进一步处理。

### BN 为何有效果
Batch Normalization 效果很好的原因有以下两点：

 - 通过对隐藏层各神经元的输入做类似的标准化处理，提高神经网络训练速度；
 - 可以使前面层的权重变化对后面层造成的影响减小，整体网络更加健壮。

关于第二点，如果实际应用样本和训练样本的数据分布不同（例如，橘猫图片和黑猫图片），我们称发生了“Covariate Shift”。这种情况下，一般要对模型进行重新训练。Batch Normalization 的作用就是减小 Covariate Shift 所带来的影响，让模型变得更加健壮，鲁棒性（Robustness）更强。

即使输入的值改变了，由于 Batch Normalization 的作用，使得均值和方差保持不变（由 γ 和 β 决定），限制了在前层的参数更新对数值分布的影响程度，因此后层的学习变得更容易一些。Batch Normalization 减少了各层 W 和 b 之间的耦合性，让各层更加独立，实现自我训练学习的效果。

另外，Batch Normalization 也起到微弱的正则化（regularization）效果。因为在每个 mini-batch 而非整个数据集上计算均值和方差，只由这一小部分数据估计得出的均值和方差会有一些噪声，因此最终计算出的 z~(i)也有一定噪声。类似于 dropout，这种噪声会使得神经元不会再特别依赖于任何一个输入特征。

因为 Batch Normalization 只有微弱的正则化效果，因此可以和 dropout 一起使用，以获得更强大的正则化效果。通过应用更大的 mini-batch 大小，可以减少噪声，从而减少这种正则化效果。

最后，不要将 Batch Normalization 作为正则化的手段，而是当作加速学习的方式。正则化只是一种非期望的副作用，Batch Normalization 解决的还是反向传播过程中的梯度问题（梯度消失和爆炸）。

### 测试的时候　BN 如何算均值和方差
测试的时候由于是一个样本一个样本测试，我们无法得到均值和方差。

理论上，我们**可以将所有训练集放入最终的神经网络模型中，然后将每个隐藏层计算得到的 $\mu^{[l]}$和 $\sigma^{[l]}$ 直接作为测试过程的 $\mu$ 和 $\sigma$ 来使用**。但是，实际应用中一般不使用这种方法，而是使用之前学习过的指数加权平均的方法来预测测试过程单个样本的 $\mu^{[l]}$和 $\sigma^{[l]}$。

对于第 l 层隐藏层，考虑所有 mini-batch 在该隐藏层下的 $\mu^{[l]}$和 $\sigma^{[l]}$，然后用指数加权平均的方式来预测得到当前单个样本的 $\mu^{[l]}$和 $\sigma^{[l]}$。这样就实现了对测试过程单个样本的均值和方差估计。

# GridSearch & RandomSearch#
遵循sk-learn的API可以自定义模型，让后可以传给 GridSearchCV 函数进行参数的搜索。
- [scikit-learn-rolling-your-own-estimator](http://scikit-learn.org/dev/developers/contributing.html#rolling-your-own-estimator)
- [how-to-create-customize-your-own-scorer-function-in-scikit-learn](https://stackoverflow.com/questions/32401493/how-to-create-customize-your-own-scorer-function-in-scikit-learn)
- [how-to-write-a-custom-estimator-in-sklearn-and-use-cross-validation-on-it](https://stackoverflow.com/questions/20330445/how-to-write-a-custom-estimator-in-sklearn-and-use-cross-validation-on-it)
- [Parameter estimation using grid search with cross-validation](http://scikit-learn.org/stable/auto_examples/model_selection/plot_grid_search_digits.html)
- [How to Grid Search Hyperparameters for Deep Learning Models in Python With Keras](https://machinelearningmastery.com/grid-search-hyperparameters-deep-learning-models-python-keras/)
- [通过嵌套交叉验证选择算法](https://ljalphabeta.gitbooks.io/python-/content/nested.html)
- [Optimal Tuning Parameters](http://www.ritchieng.com/machine-learning-efficiently-search-tuning-param/)

I am now a post graduate at USTC (University of Science and Technology of China), majored in software engineering. I have strong interests about IoT and cloud-edge intelligence, I do some research about this, and I want to get a machine learning job after my graduation. As all we know, internet of things will come soon, there will be more and more intelligent devices and it will produce more and more data. I think deep learning is one of the most powerful technics to implement the vision. It can be applied in computer vision, speech recognition, natural language processing, healthcare and so on. But now i have no enough income to pay for the course. I have only basic living allowance. I have finished 《Neural Networks and Deep Learning》and 《Improving Deep Neural Networks: Hyperparameter tuning, Regularization and Optimization》 which i have paid for $49 by cutting my living expenses. Now i really want to continue this course specification. I will be grateful if I get a scholarship.


Firstly, i have finished the 1th and 2th courses from which i have got a solid foundation about neural networks basics and optimization. I think it is time to apply it to real applications such as computer vision which is very important in intelligent edge devices.
Secondly, if i finished this course, i will get a certificate for my job hunting. Coursera courses are acknowledged by industry. This will be a good point in my resume. It will help me get more opportunities for interviews.
Thirdly, I think this course is very competitive and i will learn more from this course, which will improve my machine learning skills and i will do some research about cloud-edge intelligence more effectively. I believe i will be better at deep learning principle and application which will help me to write thesis and graduation better.
At last, this course specification will lay a solid foundation about deep learning for me. I think it will be the most valuable fortune in my career.


```python

```

# courses
 - [course.fast.ai](http://course.fast.ai/index.html)