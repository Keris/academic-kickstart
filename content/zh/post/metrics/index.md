---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "评估指标"
subtitle: ""
summary: "了解机器学习模型的评估指标"
authors: []
tags: ["评估指标"]
categories: [机器学习]
date: 2020-03-11T19:20:11+08:00
lastmod: 2020-03-11T19:20:11+08:00
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

## 混淆矩阵

|          | guessed positive     | guessed negative     |
|----------|----------------------|----------------------|
| positive | true positives, #TP  | false negatives, #FN |
| negative | false positives, #FP | true negatives, #TN  |

假设某个模型在一组样本上的表现如下：

{{< figure src="model-performance.png" numbered="true" >}}

其中，蓝色的点为正例（positives)，橙色的点为负例（negatives），则混淆矩阵如下：

|          | guessed positive | guessed negative |
|----------|------------------|------------------|
| positive | 6                | 1                |
| negative | 2                | 5                |

## Accuracy
Accuracy，也就是准确率，表示样本中分类正确所占的比例。

$$\text{Accuracy} = \frac{\text{\#TP} + \text{\#TN}}{\text{\#TP} + \text{\#FP} + \text{\#TN} + \text{\#FN}}$$

其中:

- `#TP`表示`true positive`的数目
- `#FP`表示`false positive`的数目
- `#TN`表示`true negative`的数目
- `#FN`表示`false negative`的数目

对于图1中的模型，准确率计算如下：

$$\text{Accuracy} = \frac{6 + 5}{6 + 1 + 5 + 2} = \frac{11}{14}= 78.57\\%$$

准确率在数据偏斜的情况下将不再适用。比如在下图所示的例子中：

{{< figure src="credit-card-fraud.png" numbered="true" >}}

模型需要找出良好的交易，但我们的样本中绝大部分都是良好的交易，这导致无论我们的模型如何，它都具有很高的准确率，如图中所示为`99.83%`！，此时准确率将不能够评估我们的模型。

## Precision
Precision即精度，表示在所有预测为正例的样本中有多少是真正例。

$$\text{Precision} = \frac{\text{\#TP}}{\text{\#TP} + \text{\#FP}}$$

对于图1中的模型，精度计算如下：

$$\text{Precision} = \frac{6}{6 + 2} = \frac{6}{8} = 75\\%$$

## Recall
Recall即为召回率，所有的正例样本中真正例所占的比例：

$$\text{Recall} = \frac{\text{\#TP}}{\text{\#TP} + \text{\#FN}}$$

对于图1中的模型，召回率计算如下：

$$\text{Recall} = \frac{6}{6 + 1} = \frac{6}{7} = 85.71\\%$$

## F1 score
F1-score定义如下：

$$\text{F1} = 2 \times \frac{\text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}}$$

可以看到F1-score综合了精度和召回率，为精度和召回率的调和平均：**在召回率不变的条件下，精度越高，F1-score越大；在精度不变的条件下，召回率越高，F1-score越大**。

## $F_\beta$ score
$F_\beta-\text{score}$定义为：

$$F_\beta = (1 + \beta^2) \times \frac{\text{Precision} \times \text{Recall}}{\beta^2 \times \text{Precision} + \text{Recall}}$$

可以看出来，当$\beta = 1$时即为F1 score。

下面我们看下$F_\beta$随$\beta$变化的情况：

1. 当$\beta = 0$时，通过计算不难得出$F_0 = \text{Precision}$;
2. 相反，当$\beta$取很大的值时，$F_\beta$将趋近召回率。

小结：当$\beta$取较小的值时，精度比召回率重要，如$F_{0.5}$ score；当$\beta$取较大值时召回率比精度重要，如$F_2$ score。

## ROC Curve
ROC Curve的全称为Receiver Operating Characteristic Curve，即受试者工作特性缺陷，它刻画了模型在所有分类阈值下的表现。

要绘制该曲线，我们需要计算以下两个参数：

- True Positive Rate, TPR
- False Positive Rate, FPR

TPR，表示所有的正例样本中真正例的占比，不难发现，这其实跟召回率一个概念：

$$\text{TPR} = \frac{\text{\#TP}}{\text{\#TP} + \text{\#FN}}$$

FPR，表示所有的负例样本中假正例的占比：

$$\text{FPR} = \frac{\text{\#FP}}{\text{\#FP} + \text{\#TN}}$$

在不同的分类阈值下绘制TPR vs. FPR便得到ROC曲线：

{{< figure src="ROCCurve.svg" numbered="true" >}}

## AUC
AUC的全称为Area Under a ROC Curve，即ROC曲线下的面积。

{{< figure src="AUC.png" numbered="true" >}}