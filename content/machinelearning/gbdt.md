---
title: "gbdt"
date: 2018-06-01 15:59
---

[TOC]

- [A Gentle Introduction to Gradient Boosting](http://www.ccs.neu.edu/home/vip/teach/MLcourse/4_boosting/slides/gradient_boosting.pdf)
- [XGBoost: A Scalable Tree Boosting System](https://arxiv.org/pdf/1603.02754v1.pdf)
- [GBDT算法原理与系统设计简介](http://wepon.me/files/gbdt.pdf)
- [CatBoost vs. Light GBM vs. XGBoost](https://towardsdatascience.com/catboost-vs-light-gbm-vs-xgboost-5f93620723db)
- [【转】XGBoost参数调优完全指南（附Python代码）](https://zhuanlan.zhihu.com/p/29649128)
- [机器学习算法总结--GBDT](https://blog.csdn.net/lc013/article/details/56667157)

# CTR 大框架方法
传统方法主要是线性模型如 LR,FM,FFM，提升树模型 gdbt, xgboost, lightGBM, 两种方法结合就是 facebook 的 GBDT + LR, 通过提升树自动做特征编码，然后再给LR模型。
 - [除了LR，FM（FFM）方法，CTR预测还有那些方法，应用较为广泛？](https://www.zhihu.com/question/56204961)
 - [CTR 预估模型的进化之路](https://cloud.tencent.com/developer/article/1005416)
 - [2nd place solution for Avazu click-through rate prediction competition](https://github.com/owenzhang/kaggle-avazu)
 - [kaggle-2014-criteo-champion](https://github.com/guestwalk/kaggle-2014-criteo)
 - [机器学习实践-CVR-Tencent_CVR预估初赛&复赛思路总结](https://jiayi797.github.io/2017/06/07/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E5%AE%9E%E8%B7%B5-CVR-Tencent_CVR%E9%A2%84%E4%BC%B0%E5%88%9D%E8%B5%9B&%E5%A4%8D%E8%B5%9B%E6%80%9D%E8%B7%AF%E6%80%BB%E7%BB%93/)
 - [Tencen2017_Fianl_Coda_Allegro](https://github.com/BladeCoda/Tencent2017_Final_Coda_Allegro)
 - [GBDT原理及利用GBDT构造新的特征-Python实现](https://blog.csdn.net/shine19930820/article/details/71713680)
 - [Turing Test's 3rd Place Solution for Home Depot Product Search Relevance Competition on Kaggle ](https://github.com/ChenglongChen/Kaggle_HomeDepot)
 - [Online Learning with Microsoft's AdPredictor algorithm](http://tullo.ch/articles/online-learning-with-adpredictor/)
 
梯度下降学习率更新策略，本文介绍了常见的几种，并给出的试验数据
 - 在线学习中，LR对BOPR。BOPR效果稍微好于LR，但是LR更为简单，所以最后还是选择了LR
 - GDBT迭代轮数，大部分优化只需要500轮迭代GBDT模型可以完成。
 - GDBT的数深度也不需要太深，2,3层一般满足要求。
 - 特征也不是越多越好，重要性前Top 400特征就可以表现很好
 - 历史特征比用户当前上下文特征更为重要，而且对预测的时间更为稳定。但是用户上下文数据可以有效的解决冷启动问题。
 - 无需使用全部的样本，使用10%的训练数据的效果与100%没有太大差别
 - 如果对负样本重采样，模型计算的概率，需要重新修正。修正公式为 $q = \frac{p}{p+(1-p)/w} $ 其中q是修正后的数据，p时模型预测的数据，w是负样本重采样比例。
 - 原始特征+GBDT编码特征+二阶交叉特征 要更高与单独的 GBDT编码特征 到LR
 
 
## 样本不平衡
### nagetive sampling & cabrilication
推导修正公式  $q = \frac{p}{p+(1-p)/w} $

 - [机器学习实践-不平衡类的处理](https://jiayi797.github.io/2017/06/10/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E5%AE%9E%E8%B7%B5-%E4%B8%8D%E5%B9%B3%E8%A1%A1%E7%B1%BB%E7%9A%84%E5%A4%84%E7%90%86/)

# 推荐大框架方法
 Google 的 deep & wide 模型，主要通过 embedding 和 传统特征结合，融入神经网络进行预测。
 
# 提升树模型
 catboost 表现非常强悍，比 lgbm 最高提升0.001。boosting 和 bagging。 
 
> GBDT 和 Random Forests，为啥前者比较浅，后者比较深。？

  - 对于Bagging算法来说，由于我们会并行地训练很多不同的分类器的目的就是降低这个方差(variance) ,因为采用了相互独立的基分类器多了以后, 就是在保证模型的泛化即方差，所以对于每个基分类器来说，目标就是如何降低这个偏差（bias), 所以我们会采用深度很深甚至不剪枝的决策树。
  
  - 对于Boosting来说，每一步我们都会在上一轮的基础上更加拟合原数据，所以可以保证偏差（bias）,所以对于每个基分类器来说，问题就在于如何选择variance更小的分类器，即更简单的分类器，所以我们选择了深度很浅的决策树。

计算损失函数的负梯度在当前模型的值，将它作为残差的估计,估计回归树叶节点区域，以拟合残差的近似值

> GBDT 是如何计算分类任务的残差的？
  - 提升树模型其实是使用梯度负方向替代残差，因为在分类任务中，类别加减是无意义的，另外，梯度比残差更加稳定和容错。
  - [GBDT训练分类器时，残差是如何计算的？](https://blog.csdn.net/mmc2015/article/details/52398488)
  - [机器学习算法GBDT的面试要点总结-上篇](https://www.cnblogs.com/ModifyRong/p/7744987.html)

 
# 决策树
> 决策树如何处理连续特征的?

 - 必须注意的是：根据离散特征分支划分数据集时，子数据集中不再包含该特征（因为每个分支下的子数据集该特征的取值就会是一样的，信息增益或者Gini Gain将不再变化）；而根据连续特征分支时，各分支下的子数据集必须依旧包含该特征（当然，左右分支各包含的分别是取值小于、大于等于分裂值的子数据集），因为该连续特征再接下来的树分支过程中可能依旧起着决定性作用。

 - 首先对这个连续变量排序。比如说年龄，把所有样本中年龄的数值从小到大排序。在数据没有重复的假设下，如果有n个样本，那么排序后的数据之间应该有n-1个间隔。决策树会对这n-1个间隔进行逐一尝试（分叉），每个不同的分叉，会带来不同的gini指数，我们最终选gini指数最小的那个分叉作为最优分叉。理论上是这样进行的，但是实际情况是为了一些计算优化，可能会进行一些随机搜索，而不一定是遍历。上面这个过程就把那个连续变量进行了一分为二（第一次离散化），比如说年龄被分成了0到20岁，20到100岁。接下来，当决策树继续生长时，之前一分为二的连续特征可能会再次被选中。比如说20到100岁这个分叉被选中，我们再次重复上面那三个步骤。这次得到的结果可能是20到35，35到100岁。
 
> 决策树是如何做分类和回归的？
 
   - 分类其实很简单，最常用的方式是看叶子节点中样本数据类别投票，多数者胜，同时也可以用来计算概率
 
   - 回归简单的实现方式就是叶子节点中所有样本数据目标值的平均值


> 决策树为什么容易过拟合，如何防止过拟合？
  
  - 因为决策树是在特征空间的一种划分，并且从根到叶子的每条路径其实都是一种很强的if else规则，这种规则是在训练集训练得到的，因此他往往更适用与训练集。
  - 决策树过拟合是因为树过深或者树叶过多，因此我们需要对树进行剪枝pruning. 一般会使用叶子节点数来作正则来平衡模型复杂度。带正则的优化函数一般就是两个项，一个是需要尽量准确地降低训练集误差即经验风险，一个是正则防止模型过度复杂。结构风险=经验风险+正则
  - $C_{\alpha}(T)=\sum_{t=1}^{T} N_t H_t(T) + \alpha|T|$

## 决策树的最常见实现CART
CART是最常见的决策树实现，既可以回归也可以分类，并且他是二叉树，每个节点都只会有两个分支（某个特征的阈值分割），传统决策树是多叉树（某个特征所有的分段取值都是一个分支）。

> CART 如何做分类和回归预测？
 - 和传统决策树类似，叶节点投票或者取均值
 
> CART 树分类和回归生成？
 - 分类采用gini指数
 - 回归采用最小二乘

> CART树 作为GBDT的基分类器的时候，对GBDT的分类，每个叶子节点输出的到底是什么？
 - sklearn gbdt多分类实现其实是one-vs-all,每一步提升同时训练了多个二分类树，
 
 - [scikit-learn中的GBDT实现](https://liangyaorong.github.io/blog/2017/scikit-learn%E4%B8%AD%E7%9A%84GBDT%E5%AE%9E%E7%8E%B0/)
 - [sklearn中的模型评估](http://d0evi1.com/sklearn/model_evaluation/)
## Param Tuning
 - XGBoost:
    - booster: gblinear/gbtree 基分类器用线性分类器还是决策树
    - max_depth: 树最大深度
    - learning_rate：学习率
    - alpha：L1正则化系数
    - lambda：L2正则化系数
    - subsample: 训练时使用样本的比例

 - LightGBM：
   - num_leaves: 叶子节点的个数
   - max_depth：最大深度，控制分裂的深度
   - learning_rate: 学习率
   - objective: 损失函数（mse, huber loss, fair loss等）
   - min_data_in_leaf: 叶子节点必须包含的最少样本数
   - feature_fraction: 训练时使用feature的比例
   - bagging_fraction: 训练时使用样本的比例
   
- [Using Grid Search to Optimise CatBoost Parameters](https://effectiveml.com/using-grid-search-to-optimise-catboost-parameters.html)
- [What is LightGBM, How to implement it? How to fine tune the parameters?](https://medium.com/@pushkarmandot/https-medium-com-pushkarmandot-what-is-lightgbm-how-to-implement-it-how-to-fine-tune-the-parameters-60347819b7fc)
- [Parameter Tuning in Gradient Boosting (GBM) with Python](https://www.datacareer.de/blog/parameter-tuning-in-gradient-boosting-gbm/)


# 点击率相关特征平滑
点击率、喜爱率、偏爱率的统计，如果是直接用点击数比总数，对于CTR来说往往会有偏差，比如一个用户看了一个视频，而且点了，那么他的点击率就是1，但是一个用户看了100个视频，点了80个，点击率是0.8，显然，我们不能说前者就是倾向于更爱点击了，因此我们需要做平滑处理！
雅虎一篇论文做了关于CTR特征平滑的方法，$\frac{C + \alpha}{I + \alpha + \beta}$, 关键是找到分子分母合适的平滑参数。I 代表曝光次数 impressions， C 代表点击次数 clicks

## 先验概率后验概率似然函数
 - [详解最大似然估计（MLE）、最大后验概率估计（MAP），以及贝叶斯公式的理解](https://blog.csdn.net/u011508640/article/details/72815981)
 - [先验分布、后验分布、似然估计这几个概念是什么意思，它们之间的关系是什么？](https://www.zhihu.com/question/24261751)
 

## 概率中的PDF，PMF，CDF
 - [概率中的PDF，PMF，CDF](https://blog.csdn.net/wzgbm/article/details/51680540)
 
## beta 分布、二项分布和贝叶斯推断
 beta 分布这里的应用其实是概率的概率分布，这个概率是先验概率 $p(\theta)$ 的分布假设服从 beta 分布。比如这里的视频点击概率，一个视频被点击的概率，我们先验的认为他服从 beta 分布。
 $$
 \beta(a,b) = \frac{\theta^{a-1} (1 - \theta)^{b-1}}{B(a,b)} \varpropto \theta^{a-1} (1 - \theta)^{b-1}
 $$
 
 二项分布即重复n次独立的伯努利试验。在每次试验中只有两种可能的结果，而且两种结果发生与否互相对立，并且相互独立，与其它各次试验结果无关，事件发生与否的概率在每一次独立试验中都保持不变，则这一系列试验总称为n重伯努利实验，当试验次数为1时，二项分布服从0-1分布。 而在这里，我们的视频曝光次数可以看成 n 次独立的伯努利试验，形成二项分布。二项分布的似然函数（可以认为是条件概率）：
 $$
 p(data|\theta) \varpropto \theta^C (1 - \theta)^{I-C} \\
 C = \sum_{i}{X_i}
 $$
 
 注意前面的都是人为属于概率的范畴，因为是我们假设知道概率分布，得到分布函数。后面的推断使用的是统计，反过来的方法，通过已知的假设和观测的统计数据，来计算相关参数！
 
 贝叶斯推断，贝叶斯估计的目的就是要在给定数据的情况下求出θ的值，所以我们的目的是求解如下后验概率：
 $$
 p(\theta|data) = \frac{p(data|\theta)p(\theta)}{p(data)} \varpropto p(data|\theta)p(\theta)
 $$
 以上公式就是我们常说的，**后验分布 正比于 似然函数与先验分布的乘积**. 
 
 共轭先验, 现在有了二项分布的似然函数和beta分布，将beta分布代进贝叶斯估计中的P(θ)中，将二项分布的似然函数代入P(data|θ)中，可以得到:
 $$
 p(\theta|data) \varpropto \theta^C (1 - \theta)^{I-C} \theta^{a-1} (1 - \theta)^{b-1} = \theta^{a+C-1} (1-\theta)^{b+I-C-1}
 $$
 最后的贝叶斯后验概率分布也是一个beta分布的形式，$a^{'}=a+C,b^{'}=b+I−C$ 那这里根据 beta 分布的均值 $\mu = \frac{a}{a+b}$, 可以算出 后验概率的均值 $\frac{a^{'}}{a^{'}+b^{'}} = \frac{C+a}{I+a+b}$. 而实际上，均值就是我们需要的概率最大的解，概率统计中我们学过，通过求导，取导数为0的地方即可。这个均值其实就是我们需要的点击率平滑后的值。也就是说，我们可以根据一系列的观察到的点击率 $\theta$，来估计出参数 $\alpha \beta$.
 
 现在由于我们只是假定了先验概率的分布式beta分布，但是$\theta$到底应该取哪一个值，目前需要通过其他方法学习或者推断出来，这里我们就会用到贝叶斯条件概率和极大似然法，条件概率就是执果索因，这个二项分布似然函数就是我们要最大化的地方，这就是为啥叫极大似然方法，就是我们断定在某种先验概率$\theta$（原因）给定的时候，出现这样的观测数据集的概率最大化，找到那个最大的先验概率，我们就认为找到了先验取值，从而找到了参数。对于如何找到最大似然，就是优化目标函数的问题了，可以最小二成，牛顿、梯度下降法等。 
 
 这里使用贝叶斯推断和极大似然的方法和 逻辑斯特回归模型的对数极大似然是一样的道理。

 - [如何通俗理解beta分布](https://www.zhihu.com/question/30269898)
 - [ctr平滑](http://d0evi1.com/ctr-smooth/)
 - [转化率（CTR）预测的贝叶斯平滑](https://blog.csdn.net/jinping_shi/article/details/78334362)
 - [CTR预估中的贝叶斯平滑方法（一）](https://blog.csdn.net/dengxing1234/article/details/77965536)
 - [【实践】CTR预估中的贝叶斯平滑方法（二）](https://blog.csdn.net/dengxing1234/article/details/77965657)
 - [Inferring probabilities with a Beta prior, a third example of Bayesian calculations](http://chrisstrelioff.ws/sandbox/2014/12/11/inferring_probabilities_with_a_beta_prior_a_third_example_of_bayesian_calculations.html)
 - [Introduction to Bayesian Inference](https://www.datascience.com/blog/introduction-to-bayesian-inference-learn-data-science-tutorials)
 - [第一届腾讯社交广告高校算法大赛(全国14名)](https://github.com/freelzy/Tencent_Social_Ads)
 - [yahoo-ctr-smoothing-paper](http://www.cs.cmu.edu/~xuerui/papers/ctr.pdf)


 # 数据科学竞赛中的Data Leakage
因果颠倒，比如点击之后视频就会被播放，但是视频播放并不是点击的原因，恰恰是点击的结果，因此这种特征在比赛中可以提高成绩，但是却把因果颠倒了。

 - [Data Leakage in Machine Learning](https://machinelearningmastery.com/data-leakage-machine-learning/)
 - [Working with Time Series](https://jakevdp.github.io/PythonDataScienceHandbook/03.11-working-with-time-series.html)



 # 特征离散化
离散化是特征处理的重要手段之一，可以提高模型的抗干扰能力，比如年龄20和30其实对某些视频点击差异不大，但是如果使用连续值，在LR中会导致W差异巨大。比如一个年龄突然200岁，如果可以分段，那么相对来说，就可以避免这一异常。

## 各种离散编码方式对比
 - Categorical Encoding (raw, as is)
   * 原始的分类特征，不一定是数字，比如，性别（男女），国家（中国、美国、俄罗斯）
 - Numeric Encoding
   * 分类特征分别映射到数字叫 Numeric Encoding, 只需要1维度. 比如 性别(0|1), 国家(0|1|2)
 - One-Hot Encoding
   * 进行 one hot 编码，取值种类为 N，则需要 N 个维度，因此对于像用户 id 这种上亿级别的特征，最好不适用 one hot编码
 - Binary Encoding
   * 对分类的N种取值进行二进制编码，这就是计算机编码方式，只需要 logN 的维度即可
   
LR 模型比较适合 one-hot 编码，树模型直接使用 Categorical 或者 Numerical Encoding 即可，其他编码方式对于树模型来讲，效果一般

### 参考
 - [Categorical Features and Encoding in Decision Trees](https://medium.com/data-design/visiting-categorical-features-and-encoding-in-decision-trees-53400fa65931)
 - [Are categorical variables getting lost in your random forests?](https://roamanalytics.com/2016/10/28/are-categorical-variables-getting-lost-in-your-random-forests/)
 
## ONE-HOT 编码解释
 - 使用one-hot编码，将离散特征的取值扩展到了欧式空间，离散特征的某个取值就对应欧式空间的某个点.
 - 将离散特征通过one-hot编码映射到欧式空间，是因为，在回归，分类，聚类等机器学习算法中，特征之间距离的计算或相似度的计算是非常重要的，而我们常用的距离或相似度的计算都是在欧式空间的相似度计算，计算余弦相似性，基于的就是欧式空间.
 - 将离散型特征使用one-hot编码，确实会让特征之间的距离计算更加合理。比如，有一个离散型特征，代表工作类型，该离散型特征，共有三个取值，不使用one-hot编码，其表示分别是x_1 = (1), x_2 = (2), x_3 = (3)。两个工作之间的距离是，(x_1, x_2) = 1, d(x_2, x_3) = 1, d(x_1, x_3) = 2。那么x_1和x_3工作之间就越不相似吗？显然这样的表示，计算出来的特征的距离是不合理。那如果使用one-hot编码，则得到x_1 = (1, 0, 0), x_2 = (0, 1, 0), x_3 = (0, 0, 1)，那么两个工作之间的距离就都是sqrt(2).即每两个工作之间的距离是一样的，显得更合理.
 - 离散特征进行one-hot编码后，编码后的特征，其实每一维度的特征都可以看做是连续的特征。就可以跟对连续型特征的归一化方法一样，对每一维特征进行归一化。比如归一化到[-1,1]或归一化到均值为0,方差为1.
 - 基于树的方法是不需要进行特征的归一化，例如随机森林，bagging 和 boosting等。基于参数的模型或基于距离的模型，都是要进行特征的归一化.


# 特征处理
## 类别特征和Onehot
onehot 和 embedding 特征对于训练神经网络特别有用, onehot 对于线性模型也是非常有用的，它相当于让线性模型变得非线性，通过逻辑组合特征，维度升高之后，能够实现非线性数据的划分，如果直接使用类别特征，LR可能无法分类。
>At first, onehot encoding is one of the conversion method for categorical data, and good representation of the data on neural network learning.(except random forest and other method which can use the categorical data.

- [Onehot Conversion for Categorical Data](http://www.renom.jp/notebooks/preprocessing/onehot/notebook.html)

离散特征的编码分为两种情况：
 - 离散特征的取值之间没有大小的意义，比如color：[red,blue],那么就使用one-hot编码
 - 离散特征的取值有大小的意义，比如size:[X,XL,XXL],那么就使用数值的映射{X:1,XL:2,XXL:3}

 - [第四范式](https://www.4paradigm.com/support/help/%E7%89%B9%E5%BE%81%E9%87%8D%E8%A6%81%E6%80%A7%E5%88%86%E6%9E%90.html)
 - [机器学习和统计里面的auc怎么理解？](https://www.zhihu.com/question/39840928)

# GPU Docker Image
 - [NVIDIA Docker: GPU Server Application Deployment Made Easy](https://devblogs.nvidia.com/nvidia-docker-gpu-server-application-deployment-made-easy/)
 - [nvidia/cuda](https://hub.docker.com/r/nvidia/cuda/)
 - [Docker 中玩转 GPU](https://blog.opskumu.com/docker-gpu.html)
 - [GPU-Deep Learning with Docker for noobs](https://medium.com/@flavienguillocheau/documenting-docker-with-gpu-deep-learning-for-noobs-2edd350ab2f7)
