---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "信用评分卡模型"
subtitle: ""
summary: "通过此文了信用评分卡模型的方方面面"
authors: []
tags: [评分卡]
categories: [金融风控]
date: 2020-03-03T17:28:08+08:00
lastmod: 2020-03-03T17:28:08+08:00
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

## 信用评分卡模型

信用评分卡模型是一种最常见的金融风控手段。它是指根据客户的各种属性和行为数据构建一个信用评分模型，对客户进行信用评分，并据此决定是否给予授信以及授信的额度和利率，从而识别和减少在金融交易中存在的交易风险。

以下为评分卡模型的示意图：

{{< figure src="credit-score-card-demo.png" title="Credit Score Card Demo" numbered="true" >}}


### 评分卡模型划分

在不同的业务阶段我们可以构建不同的评分卡模型。根据借贷时间，评分卡模型可以划分为以下三种：

- 贷前：申请评分卡（Application score card），又称为A卡
- 贷中：行为评分卡（Behavior score card），又称为B卡
- 贷后：催收评分卡（Collection score card），又称为C卡

### 如何利用评分卡对用户进行评分？

一个用户的评分等于基准分加上对客户各个属性的评分。对前面给出的示意图中的例子而言：

```
客户评分 = 基准分 + 年龄评分 + 性别评分 + 婚姻状况评分 + 学历评分 + 月收入评分
```

现有一客户，其年龄为27岁，性别男，已婚，本科学历，月收入为10k，那么他的评分为：

```
223(基准分) + 8(年龄评分) + 4(性别评分) + 8(婚姻状况评分) + 8(学历评分) + 13(月收入评分) = 264
```

### 小结

以上就是评分卡模型的具体用法。在前面的例子中，不难发现以下三个问题：

- 如何选择用户的属性？
- 评分卡模型采用的是对属性的分段进行评分，那么如何进行有效的分段？
- 如何对每个分段进行评分？

## 评分卡模型的开发

### 评分卡模型开发流程

信用评分卡模型的开发一般包括数据获取，EDA，数据预处理，变量筛选，模型开发及评估，生成评分卡模型，模型上线及监测。典型的开发流程图如下：

{{< figure src="credit-score-card-flowchart.png" title="Credit Score Card Flowchart" width="70%" numbered="true" >}}

#### 数据获取

数据主要有两个来源：

- 金融机构自有数据：如用户的年龄，户籍，性别，收入，负债比，在本机构的借款和还款行为等
- 第三方数据：如用户在其他机构的借贷行为，用户的消费行为数据等

#### EDA

EDA为[Exploratory Data Analysis](https://en.wikipedia.org/wiki/Exploratory_data_analysis)的简写，即探索性数据分析。这个阶段旨在了解数据的主要特性，如字段的缺失情况，异常值情况，中位数，分布等。最后制定一个合理的数据预处理方案。

#### 数据预处理

数据预处理主要包括数据清洗，变量分箱和WOE编码。

#### 数据清洗

数据清洗主要是对数据中的缺失值和异常值进行处理。一般我们选择删除缺失率超过某个阈值[^1]（如30%，50%等）的变量。

[^1]: 阈值的选择需要根据实际情况确定。

#### 变量分箱

所谓的分箱定义如下：

- 对连续变量进行分段以离散化
- 将离散变量的多个状态进行合并，减少离散变量的状态数

通过分箱操作，我们达到了对变量分段的目的。

常见的分箱类型如下：

{{< figure src="binning.png" title="常见的变量分箱类型" numbered="true" >}}

##### 无监督分箱

无监督分箱主要包括三类：

- 等频分箱：把自变量的值按从小到大排序，将自变量的取值个数等分为`k`部分，每部分作为一个分箱
- 等距分箱：把自变量的值按从小到大排序，将自变量的取值范围分为`k`个等距的区间，每个区间作为一个分箱
- 聚类分箱：用`k-means`等聚类法将自变量分为`k`类，但需要在聚类过程中保证分箱的有序性

**小结：** 无监督分箱仅考虑了各个变量自身的结构，并没有考虑自变量与目标变量之间的关系，因此这种分箱方法不一定会带来模型性能的提升。

##### 有监督分箱

有监督分箱主要包括Split分箱和Merge分箱。

1. Split分箱是一种自上而下的分箱方法，其和决策树比较相似，切分点的选择指标主要有entropy， gini指数和IV值等。
2. Merge分箱是一种自下而上的分箱方法，常见的merge分箱为ChiMerge分箱。

ChiMerge分箱的基本思想是：*如果两个相邻区间具有相似的类分布，则合并它们，否则保持不变。ChiMerge通常采用卡方值来度量两相邻区间的类分布的相似性。*

计算卡方值的公式如下：

$$
\chi^2 = \sum_{i=1}^2 \sum_{j=1}^k \frac{(A_{ij} - E_{ij})^2}{E_{ij}}
$$

其中：

$k =$ 类别数目

$A_{ij} =$ 第 $i$ 个区间中第$j$个类别的样本数目

$E_{ij} = A_{ij}$的期望频率$=\frac{R_i \times C_j}{N}$，$R_i =$第$i$区间的样本数目$= \sum_{j=1}^k A_{ij}$，$C_j = $第$j$个类别的样本数目$= \sum_{i=1}^2 A_{ij}$

ChiMerge算法包含一个初始化步和一个自底向上的合并过程。

1. 初始化：将需要离散化的属性的值按从小到大排序，将每个样本作为一个区间
2. 合并区间
    1. 计算每对相邻区间的$\chi^2$
    2. 合并具有最小$\chi^2$的相邻区间
3. 如果所有相邻区间对的$\chi^2$超过了$\chi^2-threshold$，则停止，否则回到第2步

如何确定$\chi^2-threshold$？为此我们需要选择一个想要的显著性水平和自有度，然后通过查表或者公式获得对应的$\chi^2$。其中自由度为`类别数目 - 1`，例如，当有3个类别时，自由度为2。

除了使用$\chi^2-threshold$，还可以指定`最小区间数`和`最大区间数`来确保不会产生太少或者太多的区间。

在金融风控中，当我们应用ChiMerge时，在初始化时往往有以下操作：

- 连续值按升序排列，离散值首先转化为坏客户的占比，然后再按升序排列
- 为了减少计算量，对于状态数大于某一阈值（建议为100）的变量，利用等频分箱进行粗分箱
- 若有缺失值，则缺失值单独作为一个分箱

在分箱完后还会做进一步的处理：

- 对于坏客户占比为0或者1的分箱进行合并（一个分箱内不能全为好客户或者坏客户）
- 对于样本占比超过95%的箱子进行删除
- 检查缺失分箱的坏客户比例是否和非缺失分箱相等，如果相等，进行合并

##### 小结

1. 分箱可以有效处理缺失值和异常值
2. 分箱后数据和模型会更稳定
3. 分箱可以简化逻辑回归模型，降低过拟合风险，提高泛化能力
4. 分箱将特征统一变换为类别型变量
5. 分箱后变量才可以使用标准的评分卡格式