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

假设我们找到一个矩阵$P_{k\times n}$, $P$ 可以对数据 $X_{n\times m}$ 做相应的变换操作，使得$Y_{k\times m}=P_{k\times n}X_{n\times m}$. 这样原来在n维空间的m个向量数据就变到k维空间的m个数据，$k<n$. **矩阵 $P$ 是对数据 $X$ 中的每一条列向量做变换操作，使得每一个列向量映射到新的空间。**

## 协方差
那么什么样的 P 能够再降维后还能保证原来的矩阵元素信息。 $X$ 中的是m 个列向量都是n维， 为了让这n个字段的信息尽可能保留，最后的 $Y$ 的 k 个字段也应该尽可能保留原始n维度的信息。那么 k 个维度应该非线性相关，将一组N维向量降为k维（k大于0，小于n），其目标是选择k个单位（模为1）正交基，使得原始数据变换到这组基上后，**各字段（不同特征维度）两两间协方差为0，而字段的方差（同一特征维度下不同数据）则尽可能大（在正交的约束下，取最大的k个方差）。**

数学上的协方差可以表示两组变量的关系独立性，举个例子，假设 $X$ 是二维空间的数据点。

$$
\frac{1}{m}XX^\mathsf{T}=\begin{pmatrix}
  \frac{1}{m}\sum_{i=1}^m{a_i^2}   & \frac{1}{m}\sum_{i=1}^m{a_ib_i} \\
  \frac{1}{m}\sum_{i=1}^m{a_ib_i} & \frac{1}{m}\sum_{i=1}^m{b_i^2}
\end{pmatrix}
$$

## 实对称矩阵对角化
**线性代数中，[实对称矩阵一定可以对角化](http://58.20.53.45/files/files_upload/content/material_59/content/011004/file_1.htm).** 我们的协方差矩阵恰好就是实对称矩阵.
$$
E^\mathsf{T}CE=\Lambda=\begin{pmatrix}
  \lambda_1 &             &         & \\
              & \lambda_2 &         & \\
              &             & \ddots & \\
              &             &         & \lambda_n
\end{pmatrix}
$$
因此，最后的 $Y$ 需要满足的条件的协方差矩阵 $D=YY^T$ 其实是可以通过 $C=XX^T$ 对角化来得到，而且得到的是一个满足**各字段（不同特征维度）两两间协方差为0，而字段的方差（同一特征维度下不同数据）则尽可能大（在正交的约束下，取最大的k个方差）的对角矩阵。**
$$
\begin{array}{l l l}
  D & = & \frac{1}{m}YY^\mathsf{T} \\
    & = & \frac{1}{m}(PX)(PX)^\mathsf{T} \\
    & = & \frac{1}{m}PXX^\mathsf{T}P^\mathsf{T} \\
    & = & P(\frac{1}{m}XX^\mathsf{T})P^\mathsf{T} \\
    & = & PCP^\mathsf{T}
\end{array}
$$

## PCA by  Covariance Matrix
steps:
 1. normalization matrix M.
 2. compute covariance matrix scatter matrix.
 3. compute eigen vectors and eigen values of scatter matrix.
 4. sort eigen vectors and eigen values to get sorted V.
 5. compute reduced dimension matrix MV.
 
```python
def pca(M, n_components=None, component_ids=None):
    """
    transform matrix M using the Principal Component Analysis technique.
    
    Args:
        M:
        n_components:
        components_ids:
    Return:
        reduced matrix of matrix M by PCA.
    """
    # mean of each feature
    n_samples, n_features = M.shape
    mean = np.array([np.mean(M[:, i]) for i in range(n_features)])
    # normalization
    norm_M = M - mean
    # scatter matrix
    scatter_matrix = np.dot(norm_M.T, norm_M)
    # calculate the eigenvectors and eigenvalues
    eig_val, eig_vec = np.linalg.eig(scatter_matrix)

    eig_pairs = [(np.abs(eig_val[i]), eig_vec[:, i]) for i in range(n_features)]
    # select components
    eig_pairs = sorted(eig_pairs, key=lambda x: x[0], reverse=True)
    sorted_eig_vec = np.array([ele[1] for ele in eig_pairs]).T
    
    principal_components = np.dot(norm_M, sorted_eig_vec)
#     print(principal_components)
    
    if n_components != None:
        feature = sorted_eig_vec[:, 0:n_components]
    elif component_ids != None:
        feature = sorted_eig_vec[:, component_ids]
    
    # new matrix
    matrix = np.dot(norm_M, feature)
    return matrix
```
设有m条n维数据。

1）将原始数据按列组成n行m列矩阵X

2）将X的每一行（代表一个属性字段）进行零均值化，即减去这一行的均值

3）求出协方差矩阵$C=XX^T$

4）求出协方差矩阵的特征值及对应的特征向量

5）将特征向量按对应特征值大小从上到下按行排列成矩阵，取前k行组成矩阵P

6）Y=PX即为降维到k维后的数据

## PCA by SVD
steps:
 1. normalization matrix M.
 2. extract svd basis u,s,v from M.
 3. get eigen vectors from s,v.
 4. compute reduced dimension matrix us.
 
```python
def svd_principal_components(M, n_components=None, component_ids=None):
    """
    transform matrix M using the Singular Values Decomposition technique. 
     
    Args:
        M:
        n_components:
        components_ids:
    Return:
        reduced matrix of matrix M by svd.
    """
    # mean of each feature
    n_samples, n_features = M.shape
    mean = np.array([np.mean(M[:, i]) for i in range(n_features)])
    # normalization
    norm_M = M - mean
    
    # calculate the eigenvectors and eigenvalues by SVD
    u, s, vt = la.svd(norm_M, full_matrices=False)
    s = np.diag(s)
    # select components
    if n_components != None:
        u_k = u[:,0:n_components]
        s_k = s[0:n_components, 0:n_components]
    elif component_ids != None:
        u_k = u[:, component_ids]
        rows = np.array(component_ids)
        cols = np.array(component_ids)
        s_k = s[rows[:,None], cols]
    
    # new matrix
    matrix = np.dot(u_k, s_k)
    return matrix
```

## SVD

$$
M = U\Sigma V^T, M^T = V\Sigma^TU^T, \Sigma=\Sigma^T
$$

$$
MM^T = (U\Sigma V^T)(V\Sigma U^T) = U\Sigma V^T V \Sigma^T U^T = U\Sigma \mathbb{I} \Sigma U^T = U (\Sigma^2) U^T.
$$
$$
M^TM = (V\Sigma U^T)(U\Sigma V^T) = V \Sigma^T U^T U\Sigma V^T = V\Sigma \mathbb{I} \Sigma V^T = V (\Sigma^2) V^T.
$$

SVD compute steps:
 1. we compute $M^TM$ as $v$ and $MM^T$ as $u$.
 2. compute eigen values and eigen vectors of $v$ and $u$.
 3. sort eigen values and eigen vectors.
 4. return sorted $u$, $v$, $s$.


根据上面对PCA的数学原理的解释，我们可以了解到一些PCA的能力和限制。PCA本质上是将方差最大的方向作为主要特征，并且在各个正交方向上将数据“离相关”，也就是让它们在不同正交方向上没有相关性。

因此，PCA也存在一些限制，例如它可以很好的解除线性相关，但是对于高阶相关性就没有办法了，对于存在高阶相关性的数据，可以考虑Kernel PCA，通过Kernel函数将非线性相关转为线性相关，关于这点就不展开讨论了。另外，PCA假设数据各主特征是分布在正交方向上，如果在非正交方向上存在几个方差较大的方向，PCA的效果就大打折扣了。

# 参考
 - [PCA的数学原理](http://blog.codinglabs.org/articles/pca-tutorial.html)
 - [Change of basis in Linear Algebra](https://eli.thegreenplace.net/2015/change-of-basis-in-linear-algebra/)
 - [PCASolutions](https://people.duke.edu/~ccc14/sta-663/PCASolutions.html)
 - [主成分分析（PCA）原理详解](https://zhuanlan.zhihu.com/p/37777074)
 - [Toward an exploratory medium for mathematics](http://cognitivemedium.com/emm/emm.html)
 - [svd和pca的区别和联系，附代码实现](https://mlln.cn/2017/06/28/svd%E5%92%8Cpca%E7%9A%84%E5%8C%BA%E5%88%AB%E5%92%8C%E8%81%94%E7%B3%BB%EF%BC%8C%E9%99%84%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0/)
 - [Understanding matrix factorization for recommendation (part 1) - preliminary insights on PCA](http://nicolas-hug.com/blog/matrix_facto_1)
 - [SVD-notes.pdf](http://math.mit.edu/classes/18.095/2016IAP/lec2/SVD_Notes.pdf)