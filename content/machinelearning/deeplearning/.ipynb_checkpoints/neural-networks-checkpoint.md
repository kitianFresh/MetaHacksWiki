---
title: "neural-networks"
date: 2018-02-07 21:59
---
[TOC]

# 逻辑斯特回归 (Case Study)#


task: 训练一个识别猫的LR

**输入数据的预处理:**
 - <font color='blue'>Shape and Dimension 明确训练集/测试集 数据的形状和维度，比如数量和样本特征维度 (m_train, m_test, num_px, ...)</font>
 - <font color='blue'>Reshape 输入数据shape 使其成为一个合理输入 (num_px \* num_px \* 3, 1)</font>
 - <font color="blue">"Standardize" 数据归一化处理</font>



## 1 - 模型(Model)

训练一个识别图片是否是猫的LR。一个简单的二分类器，直接使用 sigmoid 做输出层。**Logistic Regression 就是一个简单的 Neural Network!**

<img src="/static/images/ML/LR/LogReg_kiank.png" style="width:650px;height:400px;">

**模型的数学公式**:

对某个输入数据 $x^{(i)}$, 下面是公式

$$z^{(i)} = w^T x^{(i)} + b \tag{1}$$
$$\hat{y}^{(i)} = a^{(i)} = sigmoid(z^{(i)})\tag{2}$$ 


## 2 - 损失/策略(Cost)
二分类器可以直接使用 cross-entropy 交叉熵来定义损失函数:
$$ \mathcal{L}(a^{(i)}, y^{(i)}) =  - y^{(i)}  \log(a^{(i)}) - (1-y^{(i)} )  \log(1-a^{(i)})\tag{3}$$
$$ J = \frac{1}{m} \sum_{i=1}^m \mathcal{L}(a^{(i)}, y^{(i)})\tag{6}$$


## 3 - 优化/学习 算法(Algorithm)

构建神经网络的基本步骤:
1. 定义模型架构
2. 初始化模型参数
3. Loop:
    - 前向传播计算损失 (forward propagation)
    - 反向传播计算梯度 (backward propagation)
    - 使用梯度更新参数 (gradient descent)基本上全部是梯度下降法，还有牛顿法


### 3.1 - Forward and Backward propagation

Forward Propagation:
- 输入X
- 计算激活 $A = \sigma(w^T X + b) = (a^{(0)}, a^{(1)}, ..., a^{(m-1)}, a^{(m)})$
- 计算损失: $J = -\frac{1}{m}\sum_{i=1}^{m}y^{(i)}\log(a^{(i)})+(1-y^{(i)})\log(1-a^{(i)})$

简单的求导公式即 Backward Propagation: 

$$ \frac{\partial J}{\partial w} = \frac{1}{m}X(A-Y)^T\tag{7}$$
$$ \frac{\partial J}{\partial b} = \frac{1}{m} \sum_{i=1}^m (a^{(i)}-y^{(i)})\tag{8}$$


```python
def propagate(w, b, X, Y):
    
    m = X.shape[1]
    
    # FORWARD PROPAGATION (FROM X TO COST)
    A = sigmoid(np.dot(w.T, X) + b)              # compute activation
    cost = (-1. / m) * np.sum((Y*np.log(A) + (1 - Y)*np.log(1-A)), axis=1)     # compute cost
    
    # BACKWARD PROPAGATION (TO FIND GRAD)
    dw = (1./m)*np.dot(X,((A-Y).T))
    db = (1./m)*np.sum(A-Y, axis=1)

    assert(dw.shape == w.shape)
    assert(db.dtype == float)
    cost = np.squeeze(cost)
    assert(cost.shape == ())
    
    grads = {"dw": dw,
             "db": db}
    
    return grads, cost
```

### 3.2 优化算法

优化过程就是通过最小化损失函数 $J$ 来学习参数 $w$ and $b$. 对参数 $\theta$, 优化规则是 $ \theta = \theta - \alpha \text{ } d\theta$, $\alpha$ 为学习速率 learning_rate.


```python
def optimize(w, b, X, Y, num_iterations, learning_rate, print_cost = False):

    costs = []
    
    for i in range(num_iterations):
        
        
        # Cost and gradient calculation
        grads, cost = propagate(w=w, b=b, X=X, Y=Y)
        
        # Retrieve derivatives from grads
        dw = grads["dw"]
        db = grads["db"]
        
        # update rule
        w = w - learning_rate*dw
        b = b -  learning_rate*db
        
        # Record the costs
        if i % 100 == 0:
            costs.append(cost)
        
        # Print the cost every 100 training examples
        if print_cost and i % 100 == 0:
            print ("Cost after iteration %i: %f" %(i, cost))
    
    params = {"w": w,
              "b": b}
    
    grads = {"dw": dw,
             "db": db}
    
    return params, grads, costs
```

# 浅层神经网络(Shallow Neural Net) #

## 1 - 模型 Neural Network model 

神经网络中的一个隐藏层节点其实是一个逻辑斯特回归分类器。

**model**:

<img src="/static/images/ML/DL/classification_kiank.png" style="width:600px;height:300px;">


单隐藏层的神经网络可以看成是以逻辑斯特回归为基础神经节点, 再多加一层再次向前传播而已. 隐藏层的每一个单元就可以看成是一个逻辑斯特回归,
<img src="/static/images/ML/DL/nn/nn1.png" style="width:600px;height:300px;">
你可以把每个隐藏层节点都是一个逻辑斯特回归.
<img src="/static/images/ML/DL/nn/nn2.png" style="width:600px;height:300px;">
下面用公式来表示一个前向传播的过程:
<img src="/static/images/ML/DL/nn/nn3.png" style="width:600px;height:300px;">
<img src="/static/images/ML/DL/nn/nn4.png" style="width:600px;height:300px;">
整体架构：
<img src="/static/images/ML/DL/nn/nn-overview.png" style="width:600px;height:300px;">

**Mathematically**:

$$z^{[1] (i)} =  W^{[1]} x^{(i)} + b^{[1] (i)}\tag{1}$$ 
$$a^{[1] (i)} = \tanh(z^{[1] (i)})\tag{2}$$
$$z^{[2] (i)} = W^{[2]} a^{[1] (i)} + b^{[2] (i)}\tag{3}$$
$$\hat{y}^{(i)} = a^{[2] (i)} = \sigma(z^{ [2] (i)})\tag{4}$$

$$
\begin{align}
y^{(i)}_{prediction} = 
\begin{cases} 
1 & \mbox{if } a^{[2] (i)} > 0.5 \\\\
0 & \mbox{otherwise } 
\end{cases}
\end{align}\tag{5}
$$




## 2 - 损失/策略
$$
J = - \frac{1}{m} \sum\limits_{i = 0}^{m} \large\left(\small y^{(i)}\log\left(a^{[2] (i)}\right) + (1-y^{(i)})\log\left(1- a^{[2] (i)}\right)  \large  \right) \small \tag{6}
$$


## 3 - 学习/优化 算法

### 3.1 Forward and Backward Propagation 
反向传播，使用链式法则和微分进行求导得到。这里关于向量求导和标量相比，最难的部分是 shape 和 导数形式问题。求导规则和标量一样，但是需要注意点乘和转置，以及求和形式。 这里有一个Trick是根据 dW 的shape 和 dimension 来推断出导数公式。

<img src="/static/images/ML/DL/grad_summary.png" style="width:600px;height:300px;">

举个列子, 以下是 sigmoid 作为输出层构成的二分类器的交叉熵损失函数的向量形式, 改公式是 (6) 的向量形式：
$$
L = -\frac{1}{m}{Y\log(A)+ (1-Y)\log(1-A)} \qquad (7)
$$

$$
A
=
\begin{bmatrix}
   a_1 \\\\
   a_2 \\\\
   \vdots \\\\
   a_t \\\\
   \vdots \\
   a_m
\end{bmatrix} \quad
Y
=
\begin{bmatrix}
   y_1 & y_2 & \cdots & y_t & \cdots y_m
\end{bmatrix}
$$
对 A 向量中的每个元素求导

$$
d_a
=
\begin{bmatrix}
   d_{a_1} \\\\
   d_{a_2} \\\\
   \vdots \\\\
   d_{a_t} \\\\
   \vdots \\\\
   d_{a_m}
\end{bmatrix}
=
\begin{bmatrix}
   -\frac{1}{m}(\frac{y_1}{a_1}-\frac{1-y_1}{1-a_1}) \\\\
   -\frac{1}{m}(\frac{y_2}{a_2}-\frac{1-y_2}{1-a_2}) \\\\
   \vdots \\\\
   -\frac{1}{m}(\frac{y_t}{a_t}-\frac{1-y_t}{1-a_t}) \\\\
   \vdots \\\\
   -\frac{1}{m}(\frac{y_m}{a_m}-\frac{1-y_m}{1-a_m})
\end{bmatrix}
$$

以上的标量形式累积成一个向量写法之后，就是以下公式：

$$
\begin{align}
d_{a^{L}} &= \frac{\partial L}{\partial A} & \\\\
&= -\frac{1}{m}(\frac{Y^T}{A}-\frac{1-Y^T}{1-A}) & \\\\
&= \frac{1}{m}{\frac{A-Y^T}{A*(1-A)}} \qquad (3)
\end{align}
$$

其中的 $A*(1-A)$ 以及 $(A-Y^T)/A*(1-A)$ 中的 $*/$都是element-wise 的矩阵运算;

$$
\begin{align}
d_{z^{L}} &= \frac{\partial L}{\partial A}\frac{\partial A}{\partial Z} & \\\\
&= d_{a}A*(1-A) & \\\\
&= \frac{1}{m}(A-Y^T) \qquad (4)
\end{align}
$$

消除后可以化简, 这里的 A 是最后一层的激活输出;

$$
\begin{align}
d_{w^{L}} &= \frac{\partial L}{\partial Z}\frac{\partial Z}{\partial W} & \\\\
&= d_{z^{L}}(A^{L-1})^T \qquad (5)
\end{align}
$$

为了保持得到的导数矩阵的维度和$W$ 保持一致,我们必须对链式法则求导的结果运算进行转置和交换,来保证维度的一致性; 上式中 $\frac{\partial Z}{\partial W}$ 得到 $A^{L-1}$, $\frac{\partial L}{\partial Z}$ 得到 $d_z$, 由于 $d_w$ 的维度应该是 $(n_{L}, n_{L-1})$, 而 $d_z$ 的维度是 $(n_{L}, m)$, $A^{L-1}$ 的维度是 $(n_{L-1}, m)$, 因此我们推断出 $d_w = d_{z^{L}}(A^{L-1})^T$ 才合理。

最后，偏移量的导数形式很奇怪，会多出一个求和。原因是 b（如果是scalar） 的系数可以看成是全一的矩阵[[1,1,1,...1]],其系数在累计 m 个例子的误差的时候，被求和了，因此会出现 求和公式。比如 $b^{L}$ 的系数shape 是 $(n_{L}, 1)$, 而在求Z的时候，b会被广播成 $(n_L, m)$, 然后计算损失的时候会在m维度上累加m次，因此出现了 累加 m次。如果无法理解，可以直接记住这个公式。反向传播都一样。

$$
\begin{align}
d_{b^{L}} &= \frac{\partial L}{\partial Z}\frac{\partial Z}{\partial b} & \\\\
&= \sum\limits_{i = 0}^{m}{d_{z^{L}}} \qquad (5)
\end{align}
$$


```python
def backward_propagation(parameters, cache, X, Y):
    m = X.shape[1]
    
    # First, retrieve W1 and W2 from the dictionary "parameters".
    W1 = parameters["W1"]
    W2 = parameters["W2"]
        
    # Retrieve also A1 and A2 from dictionary "cache".
    A1 = cache["A1"]
    A2 = cache["A2"]
    
    # Backward propagation: calculate dW1, db1, dW2, db2. 
    dZ2= A2 - Y
    dW2 = (1/m) * np.dot(dZ2, A1.T)
    db2 = (1/m) * np.sum(dZ2, axis=1, keepdims=True)
    dZ1 = np.multiply(np.dot(W2.T, dZ2), (1 - np.power(A1, 2)))
    dW1 = (1/m) * np.dot(dZ1, X.T)
    db1 = (1/m) * np.sum(dZ1, axis=1, keepdims=True)
    
    grads = {"dW1": dW1,
             "db1": db1,
             "dW2": dW2,
             "db2": db2}
    
    return grads
```

### 3.2 参数更新公式

$\frac{\partial \mathcal{J} }{ \partial z_{2}^{(i)} } = \frac{1}{m}(a^{[2] (i)} - y^{(i)}) $

$\frac{\partial \mathcal{J} }{ \partial W_2 } = \frac{\partial \mathcal{J} }{ \partial z_{2}^{(i)} } a^{[1] (i) T} $

$\frac{\partial \mathcal{J} }{ \partial b_2 } = \sum_i{\frac{\partial \mathcal{J} }{ \partial z_{2}^{(i)}}} $

$\frac{\partial \mathcal{J} }{ \partial z_{1}^{(i)} } =  W_2^T \frac{\partial \mathcal{J} }{ \partial z_{2}^{(i)} } * ( 1 - a^{[1] (i) 2})$

$\frac{\partial \mathcal{J} }{ \partial W_1 } = \frac{\partial \mathcal{J} }{ \partial z_{1}^{(i)} }  X^T $

$\frac{\partial \mathcal{J} }{ \partial b_1 } = \sum_i{\frac{\partial \mathcal{J} }{ \partial z_{1}^{(i)}}} $

- 符号 $*$ 表示 elementwise 乘积.
- 偏导和微分等价:
    - $dW1 = \frac{\partial \mathcal{J} }{ \partial W_1 }$
    - $db1 = \frac{\partial \mathcal{J} }{ \partial b_1 }$
    - $dW2 = \frac{\partial \mathcal{J} }{ \partial W_2 }$
    - $db2 = \frac{\partial \mathcal{J} }{ \partial b_2 }$


参数更新，mini-BGD
  - $W2 = W2 - \lambda * dW2$
  - $b2 = b2 - \lambda * db2$
  - $W1 = W1 - \lambda * dW1$
  - $b1 = b1 - \lambda * db1$
  
```python
def update_parameters(parameters, grads, learning_rate = 1.2):
    # Retrieve each parameter from the dictionary "parameters"
    W1 = parameters["W1"]
    b1 = parameters["b1"]
    W2 = parameters["W2"]
    b2 = parameters["b2"]
    
    # Retrieve each gradient from the dictionary "grads"
    dW1 = grads["dW1"]
    db1 = grads["db1"]
    dW2 = grads["dW2"]
    db2 = grads["db2"]
    
    # Update rule for each parameter
    W1 = W1 - learning_rate * dW1
    b1 = b1 - learning_rate * db1
    W2 = W2 - learning_rate * dW2
    b2 = b2 - learning_rate * db2
    
    parameters = {"W1": W1,
                  "b1": b1,
                  "W2": W2,
                  "b2": b2}
    
    return parameters
```

# 深度神经网络(Deep Neural Net)

$$
\begin{align}
f(ab)&=(ab)^2 && (\text{by definition of $f$})\\\\
&=(ab)(ab)\\\\
&=a^2 b^2 && (\text{since $G$ is abelian})\\\\
&=f(a)f(b) && (\text{by definition of $f$}).
\end{align}
$$

$$
A=
\begin{bmatrix}
  \frac{1}{3} & \frac{1}{3} & \frac{1}{3} \\\\[6pt]
   \frac{2}{3} &\frac{-1}{3} &\frac{-1}{3} \\\\[6pt]
   \frac{1}{3} & \frac{1}{3} & \frac{-2}{3}
\end{bmatrix}
$$

$$
    \begin{matrix}
    1 & x & x^2 \\\\
    1 & y & y^2 \\\\
    1 & z & z^2 \\\\
    \end{matrix}
$$

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
