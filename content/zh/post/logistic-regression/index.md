---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "逻辑回归（Logistic Regression）"
subtitle: ""
summary: "理解逻辑回归中涉及的术语以及从最大似然到最小化代价函数"
authors: ["杜利强"]
tags: ["logistic-regression", "逻辑回归"]
categories: ["机器学习"]
date: 2020-01-14T16:47:22+08:00
lastmod: 2020-01-20T13:00:37+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
{{% toc %}}

## 术语
逻辑回归涉及到以下术语：

* Logistic function
* Odds
* Logit

### Logistic function
逻辑回归中的 *Logistic* 正是出于 *Logistic function* ，它是一种 *sigmoid function* ，其接受任意实值，输出一个0到1之间的值。
*标准的* logistic funtion 定义如下：

$$\sigma(z) = \frac{e^z}{e^z + 1} = \frac{1}{1 + e^{-z}}$$

如下是它在区间$[-6, 6]$之间的图像：

<a title="logistic function curve" href="https://commons.wikimedia.org/wiki/File:Logistic-curve.svg"><img width="512" alt="Logistic-curve" src="https://upload.wikimedia.org/wikipedia/commons/thumb/8/88/Logistic-curve.svg/512px-Logistic-curve.svg.png"></a>

在逻辑回归中，我们使用对数几率（log odds），**并假定它是输入特征的线性组合**：

$$z = \ln \frac{p(x)}{1 - p(x)}= \theta_0 + \theta_1 x_1 + \theta_2 x_2 = \theta^T \cdot x$$

由上式我们可以得到，

$$p(x) = \sigma(z) = \frac{1}{1 + e^{-\theta^T \cdot x}}$$

在逻辑回归模型中，这里的 $p(x)$ 为因变量在成功情形下的概率，即 $p(y=1 \mid x)$。

### Odds
如果 $p$ 表示一个事件发生的概率，那么odds定义为

$$\text{odds} = \frac{p}{1 - p}$$
也就是说，odds为发生的概率除以不发生的概率，亦可以说为成功的概率除以失败的概率。

### Logit function
Logit为Log odds, logit function 定义为 logistic function的逆，即 $g = \sigma^{-1}$。显而易见，我们有

$$g(p(x)) = \sigma_{-1}(p(x)) = \text{logit}\\,p(x) = \ln(\frac{p(x)}{1 - p(x)}) = \theta^T \cdot x$$

## 逻辑回归
逻辑回归是一个重要的机器学习算法，其目标是基于给定的数据$x$输出随机变量$y$为0或1的概率。

考虑由$\theta$参数化的线性模型，

$$h_\theta(x) = \frac{1}{1 + e^{-\theta^T \cdot x}} = \text{Pr}(y = 1 \mid x;\theta)$$

从而，$\text{Pr}(y=0 \mid x;\theta) = 1 - h_\theta(x)$。

因为$y \in \\{0, 1 \\}$，我们有

$$\text{Pr}(y \mid x;\theta) = h_\theta(x)^y (1 - h_\theta(x))^{1 - y}
$$

似然函数为

$$\begin{aligned} L(\theta \mid x) &= \Pr(Y\mid X;\theta) \\\\
&= \prod_i \Pr(y^{(i)} \mid x^{(i)};\theta) \\\\
&= \prod_i h_\theta(x^{(i)})^{y^{(i)}}(1-h_\theta(x^{(i)}))^{1-y^{(i)}} \end{aligned}$$


一般地，我们最大化对数似然函数，

$$\log L(\theta \mid x) = \sum_{i=1}^{m}\log \Pr(y^{(i)} \mid x^{(i)};\theta)$$

定义代价函数如下：

$$J(\theta) = -\frac{1}{m} \log L(\theta \mid x) = -\frac{1}{m} \sum_i^m (y^{(i)} \log h_\theta(x^{(i)}) + (1 - y^{(i)})(1 - h_\theta(x^{(i)})))$$

容易看到，我们最大化对数似然函数就是最小化代价函数 $J(\theta)$。在机器学习中，我们使用梯度下降最小化代价函数。

### 代价函数求导

下面我们对代价函数$J(\theta)$对$\theta$进行求导，我们先考虑在一个样本上的代价函数

$$J_1(\theta) = -y \log h_\theta(x) - (1 - y)(1 - h_\theta(x))$$

现在对$J_1(\theta)$对$\theta$进行求导：

注意到$\frac{d\sigma(z)}{dz} = \sigma(z) (1 - \sigma(z))$, 我们有

$$\begin{aligned}\frac{\partial}{\partial \theta_j} J_1(\theta) &= -y \frac{1}{h_\theta(x)} h_\theta(x) (1 - h_\theta(x)) x_j - (1 - y) \frac{1}{1 - h_\theta(x)} (-1) h_\theta(x) (1 - h_\theta(x)) x_j \\\\ &= (h_\theta(x) - y) x_j\end{aligned}
$$

有了以上结果，我们有

$$\frac{\partial}{\partial \theta_j} J(\theta) = \frac{1}{m}\sum_i^m \[h_\theta(x^{(i)}) - y^{(i)}\]x_j^{(i)}$$
