---
title: "PCA-SVD"
date: 2018-11-23 15:21
---

# 矩阵和向量
矩阵和向量，是线性代数里重要的课题，他们有很多运算规则和相关定理。但是单纯的数学符号和公式很难理解矩阵变换的物理和现实意义。



设向量空间 $V$ 和 该空间下的一组基向量 $S=\{ \mathbf{v_1}, \mathbf{v_2},\dots, \mathbf{v_n} \}$. 那么基$S$(basis)满足:
 1. $S$ 线性无关(linearly independent)
 2. $S$ 通过线性组合可以衍生 $V$ 中任意向量，即 $S$可以构成(span) $V$.

向量可以通过高维空间中的坐标点，其实类比二维和三维空间中有坐标系和基向量，高维空间也有类似的基向量和坐标系，n维空间一个向量$\mathbf{v}$是n维空间的一组基向量唯一线性组合而来，而组合的系数$\left[\begin{array}{c}
c_1\\
c_2\\
\vdots \\
c_n
\end{array}\right]$就是向量$\mathbf{v}$ 的坐标。

$$
\mathbf{v} = c_1\mathbf{v_1} + c_2\mathbf{v_2} + \cdots + c_n\mathbf{v_n}.
$$

## 坐标基变换

## 降维
矩阵乘于向量会得到一个新的向量，并且可以改变这个向量的维度。这个其实就是矩阵乘以向量的应用之一。矩阵作用再某个向量上会对向量进行多种变换和拉伸得到一个新的向量。线性代数还喜欢研究一种特殊的变换，就是 $Av=\lambda v$. 研究的是一个方阵的特征向量和特征值。因为这种矩阵对自己的特征向量做变换，其实就是对特征向量做线性的变换。

假设我们找到一个矩阵$P_{k\times n}$, $P$ 可以对数据 $X_{n\times m}$ 做相应的变换操作，使得$Y_{k\times m}=P_{k\times n}X_{n\times m}$. 这样原来在n维空间的m个向量数据就变到k维空间的m个数据，$k<n$


$$
\begin{array}{l l l}
  D & = & \frac{1}{m}YY^\mathsf{T} \\
    & = & \frac{1}{m}(PX)(PX)^\mathsf{T} \\
    & = & \frac{1}{m}PXX^\mathsf{T}P^\mathsf{T} \\
    & = & P(\frac{1}{m}XX^\mathsf{T})P^\mathsf{T} \\
    & = & PCP^\mathsf{T}
\end{array}
$$
那么什么样的 P 能够再降维后还能保证原来的矩阵元素信息。 $X$ 中的是m 个列向量都是n维， 为了让这n个字段的信息尽可能保留，最后的 $Y$ 的 k 个字段也应该尽可能保留 n 原始维度的信息。那么 k 个维度应该非线性相关，将一组N维向量降为K维（K大于0，小于N），其目标是选择K个单位（模为1）正交基，使得原始数据变换到这组基上后，**各字段（不同特征维度）两两间协方差为0，而字段的方差（同一特征维度下不同数据）则尽可能大（在正交的约束下，取最大的K个方差）。**

# 参考
 - [PCA的数学原理](http://blog.codinglabs.org/articles/pca-tutorial.htmls)
 - [Change of basis in Linear Algebra](https://eli.thegreenplace.net/2015/change-of-basis-in-linear-algebra/)
 - [PCASolutions](https://people.duke.edu/~ccc14/sta-663/PCASolutions.html)
 - [主成分分析（PCA）原理详解](https://zhuanlan.zhihu.com/p/37777074s)
 - [Toward an exploratory medium for mathematics](http://cognitivemedium.com/emm/emm.html)
 - [svd和pca的区别和联系，附代码实现](https://mlln.cn/2017/06/28/svd%E5%92%8Cpca%E7%9A%84%E5%8C%BA%E5%88%AB%E5%92%8C%E8%81%94%E7%B3%BB%EF%BC%8C%E9%99%84%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0/)
 - [Understanding matrix factorization for recommendation (part 1) - preliminary insights on PCA](http://nicolas-hug.com/blog/matrix_facto_1)
 - [SVD-notes.pdf](http://math.mit.edu/classes/18.095/2016IAP/lec2/SVD_Notes.pdf)