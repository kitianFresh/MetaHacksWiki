---
title: "probability"
date: 2018-08-13 17:10
---

# 蓄水池算法
> 在未知数据长度的情况下，从长度为n的数组中随机抽取k个数，但是n是未知的，不知道何时结束。

> 特例就是在数据流中随机抽取一个数，使得抽取每个数的概率均等

蓄水池算法：

假设sample数组表示抽样结果，最开始流入k个元素一次放入sample\[0,k-1\]; 对于后面的元素，假设当前流入的是第 i 个元素(i > k)，则以 $\frac{k}{i}$ 的概率留下该元素，即当产生\[1,i\] 的随机数小于等于 k 的时候，就替换掉随机数位置的元素。

下面是Python代码：
```python
import random
import copy

def reservoir_sample(data, k):
    sample = [0]*k
    for i, element in enumerate(data):
        if i < k:
            sample[i] = element
        else:
            # random.randint() 是闭区间！
            pos = random.randint(0, i)
            if pos < k: # pos < k, replace elem with data[i], or not
                sample[pos] = element
    return sample
```

测试用例：

```python
import collections
data = [1,2,3,4,5,6,7,8,9,10]
k = 1
n = 100000
counter = {}
results = []
for i in range(n):
    res = reservoir_sample(data, k)
#     res = reservior_sampling(data, k)
#     res = reservoirSampling(data, k)
    results.extend(res)
t = collections.Counter(results)
t

import matplotlib.pyplot as plt
%matplotlib inline

labels = []
sizes = []
for k, v in t.items():
    labels.append(k)
    sizes.append(v)

patches, l_text, p_text = plt.pie(sizes, labels=labels,
                                       labeldistance=1.1, autopct='%2.0f%%', shadow=False,
                                       startangle=90, pctdistance=0.6)

plt.title("Reservoir sampling")
plt.show()
```

<img src="/static/images/Bigdata/ReservoirSampleTest.png" style="width:500px;height:300px;">
<caption><center><u> <font color="purple"> **ReservoirSample** </u></font> </center></caption>

蓄水池算法证明:
要想理解这个问题，我们考虑最终被留在sample里的元素为什么或被留下，模拟这个过程就知道，即当他被加入的时候，后续步骤永远不会被替换掉。

假设1~k 个元素中任意一个元素能够存在，说明后续流入过程中，该元素都没有被替换，比如第一个位置的元素，那么当第 k+1 个元素来的时候，取不属于第一个位置的k个元素（2~k+1）即可，概率为 $\frac{k}{k+1}$， 后续以此类推，那么通过条件概率计算：
$1 \cdot [\frac{k}{k+1}\cdot \frac{k+1}{k+2} \cdots \frac{k+n-k-1}{k+n-k}] = \frac{k}{n}$

对于k+1~n 个元素中的任意一个元素，最终能够留下来的概率就是在流入的时候被选取留下，并且后续不再被替换，假设当前流入的是第 i (i > k)个元素, 首先要留下该元素的概率是在k个已经留下的元素里面任选一个位置放入该元素，概率是 $\frac{k}{i}$, 要想最后数据流结束的时候这个元素还能被留下来，且概率为 $\frac{k}{n}$，第一种想法就是强行凑出这样一种公式，这是高中数学里面很常见的一个连乘分子分母相消例子。那么条件概率计算就是：
$\frac{k}{i} \cdot [\frac{i}{i+1} \cdot \frac{i+1}{i+2} \cdots \frac{n-1}{n}] = \frac{k}{n}$

第二种想法就是，后续的数据流入过程中，每流入一个新元素，那么被留下表示事件$A$，被留下的意思是替换其他位置的元素或者不进行任何替换就表示事件，不被留下表示事件 $\bar{A}$, 即概率是 $p(\bar{A}) = \frac{1}{i+1}$, 那么$p(A)=\frac{i}{i+1}$. 以此类推。



