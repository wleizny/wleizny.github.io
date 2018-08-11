---
layout: post
title: 深入理解FM
---

### FM介绍
Factorization Machine(因子分解机，简称FM) 是由德国康斯坦茨大学 Steffen Rendle 于2010年提出，有效解决了大规模稀疏数据下的特征组合问题。
在正式学习FM前，我们先看一下二阶多项式回归模型(公式(1))。二阶多项式回归模型是在线性回归模型的基础上引入了特征的二阶组合项，实际上还是一个线性模型。为了学习参数$w_{i,j}$, 我们需要大量 $x_i, x_j$同时不为0样本。但在样本数据本身就非常稀疏的情况下，同时满足$x_i, x_j$非0的样本就非常少，这就会导致参数 $w_{i,j}$ 学习不充分，模型预测效果较差。

$$ f(x) = \underbrace{w_0 + \sum_{i=1}^{n}{w_i x_i}}_{线性模型} + \underbrace{\sum_{i=1}^{n}\sum_{j=i+1}^{n}{w_{i,j} x_i x_j}}_{交叉项} \tag{1}$$

那么，如何在稀疏样本数据下有效解决交叉项参数学习的问题，而又不严重影响模型的性能呢？FM借鉴矩阵分解的方法，给出了一个解决思路。FM使用两个隐向量 $v_i, v_j$ 的内积 $<v_i \cdot v_j>$ 来表示$w_{i, j}$。这样，就可以把公式1重写为

$$ f(x) = w_0 + \sum_{i=1}^{n}{w_i x_i} + \sum_{i=1}^{n}\sum_{j=i+1}^{n}{<v_i \cdot v_j> x_i x_j} \tag{2} $$

其中， $v_i, v_j \in R^k$, 是包含k个元素的一维向量。在公式1中，二阶交叉项参数 $w_{i, j}$ 共有 $\frac{n(n+1)}{2}$ 项，而公式2中隐向量需要学习的参数有 $nk$ 项，由于 $k$ 往往远小于 $n$， 所以转化后的模型复杂度大大降低。  
我们接着对公式(2)中的交叉项进行一些变换

$$
\begin{split}
\sum_{i=1}^{n}\sum_{j=i+1}^{n}{<v_i \cdot v_j> x_i x_j} & = \frac{1}{2} \left( \sum_{i=i}^{n} \sum_{j=1}^{n} <v_i \cdot v_j> x_i x_j - \sum_{i=1}^{n}<v_i \cdot v_i> x_i x_i \right) \\
& = \frac{1}{2} \left( \sum_{i=i}^{n} \sum_{j=1}^{n} \sum_{f=1}^{k} v_{i,f} v_{j,f} x_i x_j - \sum_{i=1}^{n} \sum_{f=1}^{k} v_{i,f} v_{i,f} x_i x_i \right) \\
& = \frac{1}{2} \sum_{f=1}^{k} \left( \sum_{i=1}^{n} \sum_{j=1}^{n} (v_{i,f} x_i)(v_{j,f} x_j) - \sum_{i=1}^{n}(v_{i,f}x_i)^2 \right) \\
& = \frac{1}{2} \sum_{f=1}^{k} \left( (\sum_{i=1}^{n} v_{i,f} x_i)^2 - \sum_{i=1}^{n}(v_{i,f}x_i)^2 \right)
\end{split} \tag{3}
$$

将3式的结果代入(2)式有

$$ f(x) = w_0 + \sum_{i=1}^{n}{w_i x_i} + \frac{1}{2} \sum_{f=1}^{k} \left( (\sum_{i=1}^{n} v_{i,f} x_i)^2 - \sum_{i=1}^{n}(v_{i,f}x_i)^2 \right) \tag{4} $$

这就是包含二阶交叉项的FM的最终形式。对各项参数求导，有

$$
\frac{\partial}{\partial\theta}f(x) =
\left\{ \begin{aligned}
& 1 & \theta = w_0 \\
& x_i & \theta = w_i \\
& x_i \sum_{j=1}^{n}v_{j,f} x_j - v_{i,f} x_i^2 & \theta = v_{i,f}
\end{aligned}
\right. \tag{5}
$$

### 损失函数
对于回归问题，我们使用均方差(Mean Square Error)来定义损失函数，即

$$ L = \frac{1}{2m}\sum_{i=1}^{m}(y - f(x))^2 + \frac{\lambda_0}{2}w_{0}^2 + \frac{\lambda_1}{2}\sum_{i=1}^{n}w_{i}^2 + \frac{\lambda_2}{2}\sum_{i=1}^{n}\sum_{f=1}^{k}v_{i,f}^2 \tag{6} $$

对于二分类问题，我们使用Logit Loss（也叫Cross-Entropy Error)来定义损失函数，假设 $y \in \{-1, 1\}$，即

$$ L = \frac{1}{m}\sum_{i=1}^{m}log(1 + e^{-y f(x)}) + \frac{\lambda_0}{2}w_{0}^2 + \frac{\lambda_1}{2}\sum_{i=1}^{n}w_{i}^2 + \frac{\lambda_2}{2}\sum_{i=1}^{n}\sum_{f=1}^{k}v_{i,f}^2 \tag{7}$$


### 使用梯度下降(Gradient Descent)求解FM
对于回归问题，对(6)式求导有
$$
\frac{\partial}{\partial\theta}L =
\left\{ \begin{aligned}
& -\frac{1}{m}\sum_{i=1}^{m}(y - f(x)) + \lambda_0w_0 & \theta = w_0 \\
& -\frac{1}{m}\sum_{i=1}^{m}(y - f(x))x_i + \lambda_1w_i & \theta = w_i \\
& -\frac{1}{m}\sum_{i=1}^{m}(y - f(x))(x_i \sum_{j=1}^{n}v_{j,f} x_j - v_{i,f} x_i^2) + \lambda_2v_{i,f} & \theta = v_{i,f}
\end{aligned}
\right. \tag{8}
$$


对于二分类问题，对(7)式求导有

$$
\frac{\partial}{\partial\theta}L =
\left\{ \begin{aligned}
& -\frac{1}{m}\sum_{i=1}^{m}y(1 - \frac{1}{1 + e^{-yf(x)}}) + \lambda_0w_0 & \theta = w_0 \\
& -\frac{1}{m}\sum_{i=1}^{m}y(1 - \frac{1}{1 + e^{-yf(x)}})x_i + \lambda_1w_i & \theta = w_i \\
& -\frac{1}{m}\sum_{i=1}^{m}y(1 - \frac{1}{1 + e^{-yf(x)}})(x_i \sum_{j=1}^{n}v_{j,f} x_j - v_{i,f} x_i^2) + \lambda_2v_{i,f} & \theta = v_{i,f}
\end{aligned}
\right. \tag{9}
$$

### 使用交替最小二乘法(Alternating Least Squares)求解FM
FM是关于参数 $\theta$ 的线性函数，因此，我们可以把FM模型重写为

$$ f(x|\Theta) = g_{(\Theta)}(x) + \theta h_{(\Theta)}(x) \tag{10}$$

即，对于 $w_0$ 有

$$ f(x|w_0) = \underbrace{\sum_{i=1}^{n}w_i x_i + \sum_{i=1}^{n}\sum_{j=i+1}^{n}w_{i,j} x_i x_j }_{g_{w_0}(x)} + w_0 \underbrace{1}_{h_{(w_0)}(x)} \tag{11} $$

对于 $w_l$ 有

$$ f(x|w_l) = \underbrace{w_0 + \sum_{i=1, i \neq l}^{n}w_i x_i + \sum_{i=1}^{n}\sum_{j=i+1}^{n}w_{i,j} x_i x_j}_{g_{(w_l)(x)}} + w_l \underbrace{x_l}_{h_{(w_l)(x)}} \tag{12} $$

对于 $v_{l,f}$ 有

$$ f(x|v_{l,f}) = \underbrace{w_0 + \sum_{i=1}^{n}w_i x_i + \sum_{i=1}^{n}\sum_{j=i+1}^{n}\sum_{p=1,p \neq f \vee l \notin \{i,j\}}^{k}v_{i,p} v_{j,p} x_i x_j }_{g_{v_{i,f}}(x)} + v_{l,f} \underbrace{x_l \sum_{i=1, i \neq l}^{n}v_{i,f}x_i}_{h_{(v_{l,f})}(x)} \tag{13} $$

则

$$ \frac{\partial}{\partial \theta} f(x|\theta) = h_{(\theta)}(x) \tag{14} $$

把式(14)带入式(8)有

$$ \frac{\partial}{\partial \theta}L = -\frac{1}{m}\sum_{i=1}^{m}(y - f(x|\theta))h_{(\theta)}(x) + \lambda_{(\theta)} \theta \tag{15} $$

我们知道，当L取到最小值时其导数必为0，因此

$$
\begin{split}
& 0 = -\frac{1}{m}\sum_{i=1}^{m}(y - f(x|\theta))h_{(\theta)}(x) + \lambda_{(\theta)} \theta \\
\Leftrightarrow & 0 = -\frac{1}{m}\sum_{i=1}^{m}(y - g_{(\theta)}(x) - \theta h_{(\theta)}(x))h_{(\theta)}(x) + \lambda_{(\theta)} \theta \\
\Leftrightarrow & \theta = -\frac{\sum_{i=1}^{m}( g_{(\theta)}(x) - y)h_{(\theta)}(x)}{\sum_{i=1}^{m}h_{(\theta)}^2(x) + m\lambda_{(\theta)}}
\end{split} \tag{16}
$$

定义 error 项

$$ e(x,y|\Theta) = f(x|\Theta) - y \tag{17} $$

则

$$\begin{split}
g_{(\theta)}(x) - y & =  f(x|\theta) - \theta h_{(\theta)}(x) - y \\
& = e(x,y|\Theta) - \theta h_{(\theta)}(x)
\end{split} \tag{18}
$$

对于每个样本 $(x,y)$，计算其对应的 error item  $e(x,y|\Theta)$, 当每次把模型参数 $\theta$ 更新为 $\theta^{*}$ 后，需要对应地更新 error item

$$ e(x,y|\Theta^*) = e(x,y|\Theta) + (\theta^* - \theta)h_{(\theta)}(x) \tag{19} $$

这里， $\Theta^*$ 包含所有模型参数，但是只有参数 $\theta$ 更新为 $\theta^*$

同样，我们也可以预先计算好 $h_{(\theta)}(x)$。对于 $w_0$ 和 $w_i$, 计算 $h_{(\theta)}(x)$ 的复杂度是 $O(1)$, 对于 $v_{l,f}$

$$\begin{split}
h_{(v_{l,f})}(x) & = x_l \sum_{i=1, i \neq l}^{n}v_{i,f}x_i \\
& = x_l \sum_{i=1}^{n}v_{i,f}x_i - v_{l,f} x_l^2
\end{split} \tag{20}
$$

定义 $q$ item

$$ q(x,y|\Theta) = \sum_{i=1}^{n}v_{i,f} x_i \tag{21} $$

则

$$ h_{(v_{l,f})}(x) = x_l q(x,y|\Theta) - v_{l,f} x_i^2  \tag{22} $$

每次更新参数 $\theta$ 后可以重新计算

$$ q(x,y|\Theta^*) = q(x,y|\Theta) + (v_{l,f}^* - v_{l,f})x_l \tag{23} $$

完整的计算过程如下

![FM_ALS_Procedure]({{ site.baseurl }}/images/fm_als_procedure.png)

### 使用马尔科夫链蒙特卡洛方法(Markov Chain Monte Carlo, MCMC)求解FM

### 使用自适应随机梯度下降(SGDA)求解FM

### 参考：
[1]	Steffen Rendle (2012): Factorization Machines with libFM, in ACM Trans. Intell. Syst. Technol., 3(3), May  
[2] Steffen Rendle, Zeno Gantner, Christoph Freudenthaler, Lars Schmidt-Thieme (2011): Fast Context-aware Recommendations with Factorization Machines, in Proceeding of the 34th  
[3] Christoph Freudenthaler, Lars Schmidt-Thieme, Steffen Rendle (2011): Bayesian Factorization Machines, in Workshop on Sparse Representation and Low-rank Approximation, Neural Information Processing Systems (NIPS-WS), Granada, Spain  
[4] Steffen Rendle (2012): Learning Recommender Systems with Adaptive Regularization, in Proceedings of the 5th ACM International Conference on Web Search and Data Mining (WSDM 2012), Seattle
