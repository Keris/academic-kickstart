---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "一个概率问题"
subtitle: ""
summary: "有些问题没有想明白那就有空再想"
authors: []
tags: [概率]
categories: [数学]
date: 2020-04-04T13:02:41+08:00
lastmod: 2020-04-04T13:02:41+08:00
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
在开始之前我想说下这个配图，它是我今天早上拍的，里面是自己做的早餐，盘中是一个香蕉切了四小段，一片吐司面包和些许腰果，而旁边的碗中是牛奶泡燕麦。这样的搭配我已经吃了一周了，我已经感受到了积极的变化，而原先我都是不吃早餐或者在下地铁后顺路买一份麦当劳。

现在我回到正题上。这个概率问题如下：

>有一个盒子，里面放了10个球，每个上面标了1到10之间的某个数字，现在进行3次有放回的抽取，问第一次的号小于第二次，第二次又小于第三次的概率是多少？

这个问题其实不难，我自认为智商中等偏下，这个题当我在面试时没有解出来:disappointed:。下面我回顾一下当初的情景。

首先，我知道样本空间$\Omega$的大小为$|\Omega| = 10^3 = 1000$，如果我们将所求事件记为$A$，并且第$i$次抽取得到的号为$N_i(i=1,2,3)$，则$A = \\{\omega = (N_1, N_2, N_3) \in \Omega: N_1 \lt N_2 \lt N_3\\}$。现在要计算的是$|A|$，即事件$A$包含了多少个点？

我采取逐个数的策略，但不是那么容易。$N_1$可以取1到8之间的数字，然后再考虑$N_2$的取值，最后是$N_3$。

$N_1 = 1$的情况:
```text
    1
   /  \
  2 ... 9
 /  \    \
3 ... 10  10
```
以上示意图中，每一层表示$N_i(i=1,2,3)$的取值，最后要计算的是叶子节点的数目。
容易计算当$N_1 = 1$时，共有$8 + 7 + \cdots + 1 = \frac{8 \cdot (8 + 1)}{2} = 36$个叶子结点。

类似地，

$N_1=2$时，有$7 + \cdots + 1 = \frac{7 \cdot (7 + 1)}{2} = 28$，

...

$N_1=8$时，有$1$。

最后的答案是$|A| = 36 + 28 + 21 + 15 + 10 + 6 + 3 + 1 = 120$，所以所求概率为$P = \frac{|A|}{|\Omega|} = \frac{120}{1000} = .12$。

以上答案我在当时的情景下限于时间以及有些许紧张也没有给出:disappointed:。

当时我还有个想法就是使用程序的方法进行计数：

```python
count = 0
for i in range(1, 9): # N1
  for j in range(i+1, 11): # N2
    for k in range(j+1, 11): # N3
      count += 1
```

以上结果也会给出正确的结果即120。这个想法在当时考虑到切屏会有不好的影响，没有实行。

后来一细想，以上两种方法都不是最优的，再三思考之后得到了最佳方法。看来自己脑子不好使了，对概率求解生疏也是原因。

下面说说怎么最优求解。

事件$A$中三个数都不同，从10个数连续3次取不同共有$10 \cdot 9 \cdot 8 = 720$，但是我们需要$N_1 \lt N_2 \lt N_3$，而三个数的排列共有$3! = 6$种，其中只有一种满足要求，所以$|A| = \frac{720}{6} = 120$。

至此，问题求解完毕。