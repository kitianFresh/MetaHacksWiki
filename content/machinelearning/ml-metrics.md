---
title: "ml-metrics"
date: 2018-09-18 23:36
---

[ROC_Analysis](http://mlwiki.org/index.php/ROC_Analysis)
[理解 ROC 和 AUC](http://vividfree.github.io/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/2015/11/20/understanding-ROC-and-AUC)
[ROC和AUC介绍以及如何计算AUC](http://alexkong.net/2013/06/introduction-to-auc-and-roc/)

# AUC 三种计算方法
## ROC曲线下面积原始定义方法
关于面积和概率理解一致的证明.[理解 ROC 和 AUC](http://vividfree.github.io/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0/2015/11/20/understanding-ROC-and-AUC)



## 简化版求面积方法
在生成ROC曲线的过程中就可以计算出来
```python
def auc(label, score):
    assert len(label) == len(score)
    data = sorted(zip(label, score), key=lambda x: x[1], reverse=True)
    auc = 0.0
    size = len(data)
    height = 0.0
    pos = [item for item in data if item[0] == 1]
    neg = [item for item in data if item[0] == 0]
    tpr = 1. / len(pos)
    fpr = 1. / len(neg)
    for item in data:
        if item[0] == 1:
            height += tpr
        else:
            auc += height * fpr
    return auc
```


## 大数据集上的抽样概率法
根据auc 描述任意取一个正例和负例，预测的整列预测值大于负例预测值的概率



"""
**According the experiments, when pca's eigen vectors are the same with the svd's right singular vectors, PCA and SVD yield identical results. or, eigen vectors have opposite sign with singular vectors, PCA and SVD yield results with opposite sign.**
"""
# def svd_principal_components(M, n_components=None, component_ids=None):
#     """
#     transform matrix M using the Singular Values Decomposition technique. 
     
#     Args:
#         M:
#         n_components:
#         components_ids:
#     Return:
#         reduced matrix of matrix M by svd.
#     """
#     # mean of each feature
#     n_samples, n_features = M.shape
#     mean = np.array([np.mean(M[:, i]) for i in range(n_features)])
#     # normalization
#     norm_M = M - mean
#     # calculate the eigenvectors and eigenvalues by SVD
#     u, s, vt = la.svd(norm_M, full_matrices=False)
#     s = np.diag(s)
#     print('----------------s------------')
#     print(s)
#     print('----------------u------------')
#     print(u)
#     print('----------------vt------------')
#     print(vt.T)
    
#     # select components
#     if n_components != None:
#         u_k = u[:,0:n_components]
#         s_k = s[0:n_components, 0:n_components]
#     elif component_ids != None:
#         u_k = u[:, component_ids]
#         rows = np.array(component_ids)
#         cols = np.array(component_ids)
#         s_k = s[rows[:,None], cols]
    
#     # new matrix
#     matrix = np.dot(u_k, s_k)
#     return matrix