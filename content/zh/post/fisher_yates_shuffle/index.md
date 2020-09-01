---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Fisher-Yates Shuffle"
subtitle: "一种等概率洗牌方法"
summary: "Fisher-Yates Shuffle"
authors: [杜利强]
tags: [shuffle, '洗牌']
categories: []
date: 2020-09-01T15:45:34+08:00
lastmod: 2020-09-01T15:45:34+08:00
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
给定一段序列，对其进行洗牌，要求每种排列的可能性一样大。这样的问题在日常工作中还是挺常见的，如测试某个排序算法的性能，对给定的输入进行洗牌获得更多的输入情况。在这篇文章中，我主要介绍一下Fisher-Yates洗牌方法，并且仅介绍其现代版本。

## 从右到左进行洗牌

这种洗牌的核心在于每次迭代时在前一半（包含当前位置）的元素中随机挑选一个，然后与当前位置的元素进行交换。

伪代码：

```
# 对n个元素的数组进行洗牌（索引为 0..n-1):
for i from n−1 downto 1 do
     j ← random integer such that 0 ≤ j ≤ i
     exchange a[j] and a[i]
```

Python 实现：

```python
import random

def shuffle_v1(source):
	n = len(source)
	for i in range(n-1, 0, -1):
		j = random.randint(0, i)
		source[i], source[j] = source[j], source[i]
```

## 从左到右进行洗牌

这种洗牌跟从右到左的方式无差别，区别在于从后一半（包含当前位置）的元素中随机选取一个，然后与当前位置的元素进行交换。

伪代码：

```
# 对n个元素的数组进行洗牌（索引为 0..n-1):
for i from 0 to n−2 do
     j ← random integer such that i ≤ j < n
     exchange a[i] and a[j]
```

## Inside-out洗牌

前面两种方法都是对数组进行原地修改，有时候我们需要返回洗牌后的结果，而保持原数据不变。为了高效，下面介绍一种在洗牌过程中包含初始化的方法。

该方法的核心在于从前i个位置随机选择一个位置j，将新数组位置j的元素放入新数组i的位置，将原始数组位置i处的元素放入新数组j的位置。

伪代码：

```
# To initialize an array a of n elements to a randomly shuffled copy of source, both 0-based:
  for i from 0 to n − 1 do
      j ← random integer such that 0 ≤ j ≤ i
      if j ≠ i
          a[i] ← a[j]
      a[j] ← source[i]
```

Python 代码：

```python
def shuffle_inside_out(source):
	n = len(source)
	a = [0] * n
	for i in range(n):
		j = random.randint(0, i)
		a[i] = a[j]
		a[j] = source[i]
	return a
```