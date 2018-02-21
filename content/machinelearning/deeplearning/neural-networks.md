---
title: "neural-networks"
date: 2018-02-07 21:59
---

# 逻辑斯特回归

<font color='blue'>
**What you need to remember:**

Common steps for pre-processing a new dataset are:
- Figure out the dimensions and shapes of the problem (m_train, m_test, num_px, ...)
- Reshape the datasets such that each example is now a vector of size (num_px \* num_px \* 3, 1)
- "Standardize" the data

## 3 - General Architecture of the learning algorithm ##

It's time to design a simple algorithm to distinguish cat images from non-cat images.

You will build a Logistic Regression, using a Neural Network mindset. The following Figure explains why **Logistic Regression is actually a very simple Neural Network!**

<img src="images/ML/LR/LogReg_kiank.png" style="width:650px;height:400px;">

**Mathematical expression of the algorithm**:

For one example $x^{(i)}$:
$$z^{(i)} = w^T x^{(i)} + b \tag{1}$$
$$\hat{y}^{(i)} = a^{(i)} = sigmoid(z^{(i)})\tag{2}$$ 
$$ \mathcal{L}(a^{(i)}, y^{(i)}) =  - y^{(i)}  \log(a^{(i)}) - (1-y^{(i)} )  \log(1-a^{(i)})\tag{3}$$

The cost is then computed by summing over all training examples:
$$ J = \frac{1}{m} \sum_{i=1}^m \mathcal{L}(a^{(i)}, y^{(i)})\tag{6}$$

**Key steps**:
In this exercise, you will carry out the following steps: 
    - Initialize the parameters of the model
    - Learn the parameters for the model by minimizing the cost  
    - Use the learned parameters to make predictions (on the test set)
    - Analyse the results and conclude

## 4 - Building the parts of our algorithm ## 

The main steps for building a Neural Network are:
1. Define the model structure (such as number of input features) 
2. Initialize the model's parameters
3. Loop:
    - Calculate current loss (forward propagation)
    - Calculate current gradient (backward propagation)
    - Update parameters (gradient descent)

You often build 1-3 separately and integrate them into one function we call `model()`.

### 4.1 - Helper functions

**Exercise**: Using your code from "Python Basics", implement `sigmoid()`. As you've seen in the figure above, you need to compute $sigmoid( w^T x + b) = \frac{1}{1 + e^{-(w^T x + b)}}$ to make predictions. Use np.exp().

</font>


# 浅层神经网络(Shallow Neural Net)

# 深度神经网络(Deep Neural Net)


# 神经网络实践核心概念
## Train/Dev/Test
## 偏差bias 和 方差 variances
## 过拟合与欠拟合
## 正则化 Regularization
## 反向随机失活 Inverted Dropout
## 数据集扩增 Data Augmentation
## 提前终止训练 Early Stop

## 数据归一化(Normalizing inputs)
## 梯度消失与爆炸(Vanishing/Exploding Gradients)
## 权重初始化(Weights Initialization)
## 梯度检查(Gradient Checking)

# 神经网络优化(Neural Network Optimization)
## BGD/SGD/mini-batch-GD
## 指数加权平均与校正(Exponetially weighted averages and bias correction)
## 动量梯度下降法(gradient descent with momentum)
## RMSprop(均方根传播)
## Adam(Adaptive momentum estimation) = Momentum + RMSprop
## 学习率衰减(Learning rate decay)


I am now a post graduate at USTC (University of Science and Technology of China), majored in software engineering. I have strong interests about IoT and cloud-edge intelligence, I do some research about this, and I want to get a machine learning job after my graduation. As all we know, internet of things will come soon, there will be more and more intelligent devices and it will produce more and more data. I think deep learning is one of the most powerful technics to implement the vision. It can be applied in computer vision, speech recognition, natural language processing, healthcare and so on. But now i have no enough income to pay for the course. I have only basic living allowance. I have finished 《Neural Networks and Deep Learning》and 《Improving Deep Neural Networks: Hyperparameter tuning, Regularization and Optimization》 which i have paid for $49 by cutting my living expenses. Now i really want to continue this course specification. I will be grateful if I get a scholarship.


Firstly, i have finished the 1th and 2th courses from which i have got a solid foundation about neural networks basics and optimization. I think it is time to apply it to real applications such as computer vision which is very important in intelligent edge devices.
Secondly, if i finished this course, i will get a certificate for my job hunting. Coursera courses are acknowledged by industry. This will be a good point in my resume. It will help me get more opportunities for interviews.
Thirdly, I think this course is very competitive and i will learn more from this course, which will improve my machine learning skills and i will do some research about cloud-edge intelligence more effectively. I believe i will be better at deep learning principle and application which will help me to write thesis and graduation better.
At last, this course specification will lay a solid foundation about deep learning for me. I think it will be the most valuable fortune in my career.

```{.python .input}

```
