---
title: "neural-networks-core"
date: 2018-02-21 22:05
---
[TOC]


# Train/Dev/Test
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
 - 100万数据: 98%/1%/1%
 - 百万以上：　99.5%/0.25%/0.25%

## 注意
**验证集要和训练集来自于同一个分布**（数据来源一致），可以使得机器学习算法变得更快并获得更好的效果。
如果不需要用无偏估计来评估模型的性能，则可以不需要测试集

# 偏差bias 和 方差 variances
## 偏差-方差分解
“偏差-方差分解”（bias-variance decomposition）是解释学习算法泛化性能的一种重要工具。

泛化误差可分解为偏差、方差与噪声之和：
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
逻辑斯特回归的正则化例子
$$
J(w,b) = \frac{1}{m}\sum_{i=1}^mL(\hat{y}^{(i)},y^{(i)})+\frac{\lambda}{2m}{||w||}^2_2
$$
 - L2
 $$
 \frac{\lambda}{2m}{||w||}^2_2 = \frac{\lambda}{2m}\sum_{j=1}^{n_x}w^2_j = \frac{\lambda}{2m}w^Tw
 $$
 - L1
 $$
 \frac{\lambda}{2m}{||w||}_1 = \frac{\lambda}{2m}\sum_{j=1}^{n_x}{|w_j|}
 $$
神经网络正则化例子
成本函数：
$$
J(w^{[1]}, b^{[1]}, ..., w^{[L]}, b^{[L]}) = \frac{1}{m}\sum_{i=1}^mL(\hat{y}^{(i)},y^{(i)})+\frac{\lambda}{2m}\sum_{l=1}^L{{||w^{[l]}||}}^2_F
$$
$w^{[l]} shape = (n^{[l-1]}, n^{[l]})$. 有如下公式：
$$
{{||w^{[l]}||}}^2_F = \sum^{n^{[l-1]}}_{i=1}\sum^{n^{[l]}}_{j=1}(w^{[l]}_{ij})^2
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

\approx{d\theta[i]} = \frac{\partial J}{\partial \theta_i}
$$
梯度检查:
$$
\frac{{||d\theta_{approx} - d\theta||}_2}{{||d\theta_{approx}||}_2+{||d\theta||}_2}
$$

梯度检查注意事项:
 - 不要在训练中使用梯度检验，它只用于调试（debug）。使用完毕关闭梯度检验的功能；
 - 如果算法的梯度检验失败，要检查所有项，并试着找出 bug，即确定哪个 dθapprox[i] 与 dθ 的值相差比较大；
 - 当成本函数包含正则项时，也需要带上正则项进行检验；
 - 梯度检验不能与 dropout 同时使用。因为每次迭代过程中，dropout 会随机消除隐藏层单元的不同子集，难以计算 dropout 在梯度下降上的成本函数 J。建议关闭 dropout，用梯度检验进行双重检查，确定在没有 dropout 的情况下算法正确，然后打开 dropout；

# 神经网络优化(Neural Network Optimization)
## BGD/SGD/mini-batch-GD
## 指数加权平均与校正(Exponetially weighted averages and bias correction)
## 动量梯度下降法(gradient descent with momentum)
## RMSprop(均方根传播)
## Adam(Adaptive momentum estimation) = Momentum + RMSprop
## 学习率衰减(Learning rate decay)


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