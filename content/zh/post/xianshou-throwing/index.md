---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "先手优势"
subtitle: "投掷一枚硬币，先出现正面者为赢，先手好还是后手？"
summary: ""
authors: [杜利强]
tags: ["先手", "概率"]
categories: [""]
date: 2020-03-24T14:00:50+08:00
lastmod: 2020-03-24T14:00:50+08:00
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
棋经有云，“宁失一子，勿失一先”，可见先手的好处所在。为了建立对先手直观的理解，我们来玩一个游戏：

*两个人玩投掷硬币的游戏，先掷出正面的人获胜，那么你是选择先掷还是后掷？*

## 数学解法
假设硬币是公平的，即出现正面和反面的概率一样，均为`1/2`。我们用`H`表示正面朝上，`T`表示反面朝上。那么，在先手下获胜的情况是第`n`次掷得正面，前`n-1`次都为反面，其中`n`为奇数。所以先手获胜的概率为

$$p(\text{win}|\text{first}) = \sum_{k=0}^{\infty}(\frac{1}{2})^{2k + 1} = \frac{2}{3}$$

相反，在后手下获胜的情况是第`n`次掷得正面，前`n-1`次均为反面，其中`n`为偶数。所以，后手获胜的概率为

$$p(\text{win}|\text{second}) = \sum_{k=1}^{\infty}(\frac{1}{2})^{2k} = \frac{1}{3}$$

可以发现，先手获胜的概率为2/3，而后手获胜的概率为1/3，这告诉我们在生活中抓住先机，主动出击往往更能取得胜利。

## 模拟法
前面我们用数学的方法进行了求解，下面来看看如何用程序来模拟，为此我写了一个Python程序：

```python
# coding: utf-8
# filename: code.py
# Author: Liqiang Du <keris.du@gmail.com>
import random
import numpy as np


def simulate(n_games):
    """
    Simulate coin throwing game, the game is over when head appears.

    Args:
        n_games (int): the number of games.

    Returns:
        tuple: the first is the probability of win when first play,
        and the second the probability of win when second play.
    """
    win_count = np.array([0, 0])
    for i in range(n_games):
        game_over = False
        n_tails = 0
        while not game_over:
            x = random.randint(0, 1)  # 1 indicates head and 0 tail
            if x == 1:
                game_over = True
            else:
                n_tails += 1
        win_count[n_tails % 2] += 1
    return win_count.astype(float) / n_games


if __name__ == "__main__":
    n_games = 100000
    probs = simulate(n_games)
    print(probs)
```
程序的核心部分在于`simulate`，每次游戏，出现正面游戏就结束，同时记录反面出现的次数，如果该次数为偶数，则先手为赢，否则后手为赢。最后程序返回一个元祖，第一个表示先手获胜的概率，第二个为后手获胜的概率。

我们运行一下程序看看结果：
```
$ python code.py
[0.66436 0.33564]
```
我们模拟了10万次游戏，程序返回的结果跟数学求解结果十分接近。

好了，文章到此就完了。这里我通过一个简单的游戏说明了先手的好处，这告诉我们一个道理，主动出击获得先手优势往往更能够成功。