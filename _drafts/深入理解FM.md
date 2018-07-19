---
layout: post
title: 深入理解FM
---
Factorization Machine(因子分解机，简称FM) 是由德国康斯坦茨大学 Steffen Rendle 于2010年最早提出，有效解决了大规模稀疏数据下的特征组合问题。
在正式学习FM前，我们先看一下二阶多项式回归模型(公式1)。二阶多项式回归模型是在线性回归模型的基础上引入了特征的二阶组合项，实际上还是一个线性模型。为了学习参数$w_{i,j}$, 我们需要大量 $x_i, x_j$同时不为0样本。但在样本数据本身就非常稀疏的情况下，同时满足$x_i, x_j$非0的样本就非常少，这就会导致参数$w_{i,j}$学习不充分，模型预测效果较差。

$$ y = \underbrace{w_0 + \sum_{i=1}^{n}{w_i x_i}}_{线性模型} + \underbrace{\sum_{i=1}^{n}\sum_{j=i+1}^{n}{w_{i,j} x_i x_j}}_{交叉项} \tag{1}$$

那么，如何在稀疏样本数据下有效解决交叉项参数学习的问题，而又不严重影响模型的性能呢？FM借鉴矩阵分解的方法，给出了一个解决思路。

### 参考：
[1]	[Steffen Rendle (2012): Factorization Machines with libFM, in ACM Trans. Intell. Syst. Technol., 3(3), May](http://doi.acm.org/10.1145/2168752.2168771)
