---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "模型选择"
subtitle: ""
summary: "如何选择一个好的机器模型，欠拟合和过拟合判断"
authors: []
tags: []
categories: []
date: 2020-03-13T19:14:06+08:00
lastmod: 2020-03-13T19:14:06+08:00
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

## 模型选择

### 两种类型的错误
在选择模型时我们可能犯两种类型的错误，选择过于简单的模型或者过于复杂的模型，前者对应**欠拟合**，后者对应**过拟合**。

欠拟合发生时，模型具有高偏差（high bias），模型不能很好地拟合训练集；过拟合发生时，模型具有高方差（high variance），模型试图对训练数据进行记忆而不是学习其特点，在训练集上表现很好，但在测试集上表现很差。

下面我们看一个欠拟合的例子：

{{< figure src="underfitting.png" >}}

在这个例子中，左边的模型使用一个二次曲线去拟合数据点，而右边则是使用直线去拟合，可以发现直线并不能很好地拟合数据，这就是欠拟合的情形。

了解了欠拟合，我们来看一个过拟合的例子：

{{< figure title="Overfitting" src="overfitting.png" >}}

在这个例子中，左边的模型使用一个二次曲线去拟合数据点，而右边的模型试图对训练集进行记忆，因而它在训练集上表现很好，但它没有学习到数据的良好属性以很好地泛化到测试集，这就是过拟合的情形。

前面两个例子属于回归模型，在分类模型中我们也会看到欠拟合和过拟合。

分类模型中的欠拟合：

{{< figure title="Underfitting in a Classification Model" src="underfitting-classification.png" >}}

分类模型中的过拟合：

{{< figure title="Overfitting in a Classification Model" src="overfitting-classification.png" >}}

最后，我们总结一下：

{{< figure title="Tradeoff between High Bias and High Variance" src="tradeoff-high-bias-high-variance.png" >}}

### 模型复杂图
在前面一部分，我们学习了欠拟合和过拟合，在这一部分，我们了解一下模型复杂图，即评估指标随着模型复杂性的提升如何变化。

{{< figure title="Model Complexity Graph" src="model-complexity-graph.png" >}}

在上面这个图中，左侧对应欠拟合的情形，此时模型在训练集和验证集上都具有较大的误差；右侧对应过拟合的情形，模型在训练集上具有较小的误差，而在验证集上具有较大的误差；圈起来的那个地方对应刚刚好的情形，此时模型在训练集和验证集上都具有较小的误差，过了这个点，在训练集上的误差持续减小，但在验证集上的误差开始不断变大。

可能你注意到了一个新概念，**验证集**，为什么不用测试集？

在构建机器学习模型时，一个务必要坚持的原则是**测试集只能用于最后的模型评估，而不能参与模型训练**。那么我们怎么选择一个好的模型，验证集即用于此目的。

那怎么得到验证集？下图可以给你一个直观的理解：

{{< figure title="Cross Validation" src="cross-validation.png" >}}

可以看到，训练集中的一小部分被用来作为验证集，借助验证集我们评估模型是否欠拟合，过拟合或者刚刚好，即验证集用来进行决策，帮助我们选择一个好的模型。最后，测试集用于最终的模型评估。

### K折交叉验证

可能你注意到了，由于从训练集中划出了一小部分作为验证集，这使得训练数据变少了，这会对模型产生不利因素，为此你需要使用K折交叉验证。

{{< figure src="k-fold-1.png" >}}

在上面这个图中，我们有12个样本，其中实心的用于训练，空心的用于测试[^1]。这是一个4折交叉验证，每次其中一折用于测试，剩余三折用于训练。第一次，第一折样本`[0,1,2]`用于测试；第二次，第二折样本`[3,4,5]`用于测试；第三次，第三折样本`[6,7,8]`用于测试；最后一次，最后一折样本`[9,10,11]`用于测试。 可以看到，经过4折交叉验证，**训练集中的所有样本都参与了模型训练**。

[^1]: 这里的测试实为验证，不同于最终的测试，后者指模型评估，前者为模型验证。

接下来，我们看看如何在`sklearn`中进行K折交叉验证：

```python
import numpy as np
from sklearn.model_selection import KFold
kf = KFold(4)
X = np.arange(12)
for train_indices, test_indices in kf.split(X):
    print(train_indices, test_indices)
```

以上代码产生如下结果：

```
[ 3  4  5  6  7  8  9 10 11] [0 1 2]
[ 0  1  2  6  7  8  9 10 11] [3 4 5]
[ 0  1  2  3  4  5  9 10 11] [6 7 8]
[0 1 2 3 4 5 6 7 8] [ 9 10 11]
```
可以看到，输出结果跟图中的例子完全一致。可能你也注意到了，在这个例子中，每一折都是顺序产生，其实我们可以进行随机选取。

{{< figure src="k-fold-2.png" >}}
同样地，我们看看在`sklearn`中如何做：

```python
import numpy as np
from sklearn.model_selection import KFold
kf = KFold(4, shuffle=True)
X = np.arange(12)
for train_indices, test_indices in kf.split(X):
    print(train_indices, test_indices)
```
以上代码的结果如下：

```
[0 1 2 3 4 6 7 8 9] [ 5 10 11]
[ 0  1  2  4  5  6  7 10 11] [3 8 9]
[ 0  1  3  4  5  8  9 10 11] [2 6 7]
[ 2  3  5  6  7  8  9 10 11] [0 1 4]
```
可以看到，每一折中的样本不再连续，而是随机抽取的。

### 学习曲线
在模型的训练过程中，通过绘制训练误差和验证误差 vs. 参与训练的样本数，我们可以得到两条曲线。

下面我们来看一个图：

{{< figure title="Learning Curves" src="learning-curves-1.png" >}}

图的左侧是一个高偏差的模型（*对应前述的线性模型*），即模型欠拟合。在训练样本比较少时，模型能很好地拟合数据，因此训练误差很小，但当评估模型时，由于训练样本较少，模型的表现不会太好，因此验证误差会很大。随着参与训练的样本增加，更多的样本需要拟合，拟合难度增加，训练误差可能会增大一点。此时，由于使用了更多的训练数据，我们会得到一个稍微好点的模型，因此验证误差会减小一点，但不会太多。最后，当更多的数据加入训练，训练误差会增加一点，验证误差会减小一点，它们会越来越靠近。

图的中间一个刚刚好的模型（*对应二次曲线模型*）。跟前面描述的情况一样，随着参与训练的样本增加，训练误差会增加，验证误差会减小，最后它们会越来越接近，不同于左侧的情况，**验证误差和训练误差会收敛到比较低的位置**。

图的右侧是一个高方差的模型（*对应高阶拟合模型*），一开始的情况跟前面两个模型的情况一样，但随着参与训练的样本增加，训练误差会增加，但会比较小，因为此时的模型试图对训练集进行记忆，同时验证误差会减小，但会比较大。最后，**验证误差和训练误差会收敛，但不会靠近**。

接下来，我们看一个具体的例子：

样本包含两类数据，如下所示：

{{< figure title="Scatter Plot of data" src="data-visualization.png" >}}

针对该数据，我们拟合以下三个模型，看看它们的学习曲线如何：

- 逻辑回归模型
- 决策树模型
- 支持向量机模型

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.model_selection import learning_curve
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.svm import SVC

# Load data
df = pd.read_csv('data.csv')
X = np.array(df[['x1', 'x2']])
y = np.array(df['y'])

# Fix random seed
np.random.seed(55)

def randomize(X, Y):
    permutation = np.random.permutation(Y.shape[0])
    X2 = X[permutation,:]
    Y2 = Y[permutation]
    return X2, Y2

X2, y2 = randomize(X, y)

# Construct three models to draw learning curves
estimators = {}

# Logistic Regression
estimators['LR'] = LogisticRegression()

# Decision Tree
estimators['DT'] = GradientBoostingClassifier()

# Support Vector Machine
estimators['SVC'] = SVC(kernel='rbf', gamma=1000)

plt.figure(figsize=(12, 6))

for i, m in enumerate(estimators):
    train_sizes, train_scores, test_scores = learning_curve(
        estimators[m], X2, y2, cv=None, n_jobs=1, train_sizes=np.linspace(.1, 1.0, 3))
    train_scores_mean = np.mean(train_scores, axis=1)
    # train_scores_std = np.std(train_scores, axis=1)
    test_scores_mean = np.mean(test_scores, axis=1)
    # test_scores_std = np.std(test_scores, axis=1)

    plt.subplot(1, 3, i + 1)
    plt.grid()

    plt.title("Learning Curves of {}".format(m))
    plt.xlabel("Training examples")
    plt.ylabel("Score")

    plt.plot(train_scores_mean, 'o-', color="g",
             label="Training score")
    plt.plot(test_scores_mean, 'o-', color="y",
             label="Cross-validation score")


    plt.legend(loc="best")

plt.tight_layout()
plt.show()
```
以上代码将生成三个模型的学习曲线，我们来看一看：

{{< figure src="learning-curves.png" >}}

这里我们要注意的一个地方是图中使用了得分而不是误差，得分跟误差恰好相反，所以在看图时我们要在脑海里试着将其倒过来。

根据前面讲的不难分析出，决策树模型刚刚好，逻辑回归模型欠拟合，而支持向量机模型过拟合。
比如对支持向量机模型而言，模型在训练集上一直具有很高的得分（接近1），这意味着模型能够很好的拟合训练集，因此具有很小的训练误差，但在验证集上，得分一开始就不高，接着是减小，后趋于不变，这意味验证误差从一个较小值变大，然后收敛。训练得分和验证得分最后收敛，但不靠近。