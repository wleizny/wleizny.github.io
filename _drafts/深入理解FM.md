---
layout: post
title: 深入理解FM
---
Factorization Machine(因子分解机，简称FM) 是由德国康斯坦茨大学 Steffen Rendle 于2010年提出，有效解决了大规模稀疏数据下的特征组合问题。
在正式学习FM前，我们先看一下二阶多项式回归模型(公式1)。二阶多项式回归模型是在线性回归模型的基础上引入了特征的二阶组合项，实际上还是一个线性模型。为了学习参数$w_{i,j}$, 我们需要大量 $x_i, x_j$同时不为0样本。但在样本数据本身就非常稀疏的情况下，同时满足$x_i, x_j$非0的样本就非常少，这就会导致参数 $w_{i,j}$ 学习不充分，模型预测效果较差。

$$ y = \underbrace{w_0 + \sum_{i=1}^{n}{w_i x_i}}_{线性模型} + \underbrace{\sum_{i=1}^{n}\sum_{j=i+1}^{n}{w_{i,j} x_i x_j}}_{交叉项} \tag{1}$$

那么，如何在稀疏样本数据下有效解决交叉项参数学习的问题，而又不严重影响模型的性能呢？FM借鉴矩阵分解的方法，给出了一个解决思路。FM使用两个隐向量 $v_i, v_j$ 的内积 $<v_i \cdot v_j>$ 来表示$w_{i, j}$。这样，就可以把公式1重写为

$$ y = w_0 + \sum_{i=1}^{n}{w_i x_i} + \sum_{i=1}^{n}\sum_{j=i+1}^{n}{<v_i \cdot v_j> x_i x_j} \tag{2} $$

其中， $v_i, v_j \in R^k$, 是包含k个元素的一维向量。在公式1中，二阶交叉项参数 $w_{i, j}$ 共有 $\frac{n(n+1)}{2}$ 项，而公式2中隐向量需要学习的参数有 $nk$ 项，由于 $k$ 往往远小于 $n$， 所以转化后的模型复杂度大大降低。
我们接着对公式2中的交叉项进行一些变换

$$
\begin{split}
\sum_{i=1}^{n}\sum_{j=i+1}^{n}{<v_i \cdot v_j> x_i x_j} & = \frac{1}{2} \left( \sum_{i=i}^{n} \sum_{j=1}^{n} <v_i \cdot v_j> x_i x_j - \sum_{i=1}^{n}<v_i \cdot v_i> x_i x_i \right) \\
& = \frac{1}{2} \left( \sum_{i=i}^{n} \sum_{j=1}^{n} \sum_{f=1}^{k} v_{i,f} v_{j,f} x_i x_j - \sum_{i=1}^{n} \sum_{f=1}^{k} v_{i,f} v_{i,f} x_i x_i \right) \\
& = \frac{1}{2} \sum_{f=1}^{k} \left( \sum_{i=1}^{n} \sum_{j=1}^{n} (v_{i,f} x_i)(v_{j,f} x_j) - \sum_{i=1}^{n}(v_{i,f}x_i)^2 \right) \\
& = \frac{1}{2} \sum_{f=1}^{k} \left( (\sum_{i=1}^{n} v_{i,f} x_i)^2 - \sum_{i=1}^{n}(v_{i,f}x_i)^2 \right)
\end{split} \tag{3}
$$

将3式的结果代入2式有

$$ y = w_0 + \sum_{i=1}^{n}{w_i x_i} + \frac{1}{2} \sum_{f=1}^{k} \left( (\sum_{i=1}^{n} v_{i,f} x_i)^2 - \sum_{i=1}^{n}(v_{i,f}x_i)^2 \right) \tag{4} $$

这就是FM的最终形式。我们对参数求导有

$$
\frac{\partial}{\partial\theta}y =
\left\{ \begin{aligned}
& 1 & \theta = w_0 \\
& x_i & \theta = w_i \\
& x_i \sum_{i=1}^{n}v_{i,f} x_i - v_{i,f} x_i^2 & \theta = v_{i,f}
\end{aligned}
\right. \tag{5}
$$

带正则化项的FM模型

$$ y = w_0 + \sum_{i=1}^{n}{w_i x_i} + \frac{1}{2} \sum_{f=1}^{k} \left( (\sum_{i=1}^{n} v_{i,f} x_i)^2 - \sum_{i=1}^{n}(v_{i,f}x_i)^2 \right) + \frac{1}{2}\alpha_0w_o^2 + \frac{1}{2}\sum_{i=1}^{n}\alpha_1w_i^2 + \frac{1}{2}\sum_{i=1}^{n}\sum_{f=1}^{k}\alpha_2v_{i,f}^2 \tag{6} $$

对参数求导有

$$
\frac{\partial}{\partial\theta}y =
\left\{ \begin{aligned}
& 1 + \alpha_0 & \theta = w_0 \\
& x_i + \alpha_1 & \theta = w_i \\
& x_i \sum_{i=1}^{n}v_{i,f} x_i - v_{i,f} x_i^2 + \alpha_2 & \theta = v_{i,f}
\end{aligned}
\right. \tag{5}
$$

### 参考：
[1]	[Steffen Rendle (2012): Factorization Machines with libFM, in ACM Trans. Intell. Syst. Technol., 3(3), May](http://doi.acm.org/10.1145/2168752.2168771)
