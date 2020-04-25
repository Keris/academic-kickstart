---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "简单粗暴正则表达式"
subtitle: "在30分钟内了解正则表达式的方方面面"
summary: ""
authors: []
tags: [正则表达式,Python]
categories: []
date: 2020-04-22T14:50:10+08:00
lastmod: 2020-04-22T14:50:10+08:00
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
在工作和学习中我时不时会使用正则表达式处理一些文本分析任务，但仍觉得自己是个新手，熟手也算不上。但从现在开始，我想做个这方面的能手，写这篇文章即为开始，也希望能帮助到需要的朋友。写了这篇文章也不意味着我就成能手了，要专于此，路且长漫。俗话说，共欲善其事必先利其器，正则表达式就是这样一个值得利的工具，这篇文章试图提供一个短小简练的正则表达式参考手册。

{{% toc %}}

## 什么是正则表达式

正则表达式（简称为*regex*）是一个由字符和特殊符号组成的字符串模式，它描述了一系列符合某个句法规则的字符串。正则表达式中的特殊符号称之为元字符。以下举几个例子来说明。

>搜索 v.s. 匹配
>
>匹配指的是判断一个字符串能否从开始位置全部或者部分地匹配某个模式。
>搜索指的是在目标字符串的任意位置开始搜索匹配的模式。

| 正则表达式模式 | 匹配的字符串 |
|----------------|--------------|
| foo            | foo          |
| Python         | Python       |
| abc123         | abc123       |

## 正则表达式有什么用

正则表达式是用来处理字符串的，想知道其有何用，不妨看看它的应用：

* 许多文本编辑器、IDEs支持以正则表达式进行查找和替换
* 许多编程语言支持正则表达式，如C++，Perl，Python，SQL

{{< figure src="vscode-find-date.png" title="在Visual Studio Code中查找本文档里包含的日期" numbered="true" >}}


## 正则表达式的构成

前面已经提到正则表达式由字符和特殊符号组成。

* 符号

| 表示法       | 描述                                | 正则表达式示例            |
|--------------|-----------------------------------|---------------------------|
| `literal`    | 匹配文本字符串中的字面值*literal*   | `foo`                     |
| `re1|re2`    | 匹配正则表达式re1或者re2            | `foo|bar`                 |
| `.`          | 匹配任何字符（除了`\n`）              | `b.b`                     |
| `^`          | 匹配字符串开始部分                  | `^Dear`                   |
| `$`          | 匹配字符串终止部分                  | `/bin/.*sh$`              |
| `*`          | 匹配0次或者多次前面的正则表达式     | `[A-Za-z0-9]*`            |
| `+`          | 匹配1次或者多次前面的正则表达式     | `[a-z]+\.com`             |
| `?`          | 匹配0次或者1次前面的正则表达式      | `goo?`                    |
| `{n}`        | 匹配n次前面的正则表达式             | `[0-9]{3}`                |
| `{m,n}`      | 匹配m~n次前面的正则表达式           | `[0-9]{5,9}`              |
| `[...]`      | 匹配来自字符集的任意单一字符        | `[aeiou]`                 |
| `[..x-y..]`  | 匹配x~y范围中的任意单一字符         | `[0-9], [a-zA-Z]`         |
| `[^...]`     | 不匹配来自字符集的任一字符          | `[^aeiou], [^0-9A-Za-z]` |
| `(*|+|?{})?` | 非贪婪版本                          | `.*?[a-z]`                |
| `(...)`      | 匹配封闭的正则表达式，然后另存为子组 | `([0-9]{3})?, f(oo|u)bar` |

* 特殊符号

>以下特殊字符（**前四个**）的大写形式对应相反的含义。

| 表示法   | 描述                  | 正则表达式示例 |
|----------|---------------------|----------------|
| `\d`     | 等同于`[0-9]`         | `data\d+.txt`  |
| `\w`     | 等同于`[0-9a-zA-Z_]`  | `[A-Za-z]\w+`  |
| `\s`     | 等同于`[\n\t\r\v\f]`  | `of\sthe`      |
| `\b`     | 匹配任何单词边界      | `\bThe\b`      |
| `\N`     | 反向引用已保存的子组  | `price;\1`     |
| `\c`     | 逐字匹配任何特殊字符c | `\.,\\,\*`     |
| `\A(\Z)` | 等同于`^($)`          | `\ADear`       |

* 扩展表示法

| 表示法            | 描述                                                               | 正则表达式示例    |
|-------------------|------------------------------------------------------------------|-------------------|
| `(?iLmsux)`       | 在正则表达式中嵌入一个或多个特殊“标记”参数                         | `(?x), (? im)`    |
| `(?:...)`         | 表示一个匹配不用保存的分组                                         | `(?:\w+\.)*`      |
| `(?P<name>...)`   | 表示仅由name标识而不是数字ID标识的正则分组匹配                     | `(?P<data>)`      |
| `(?P=name)`       | 在同一个字符串中匹配由`(?P<name>)`分组的之前的文本                 | `(?P=data)`       |
| `(?#...)`         | 表示注释                                                           | `(?#comment)`     |
| `(?=...)`         | 零宽度正预测先行断言                                               | `(?=\.com)`       |
| `(?!...)`         | 零宽度负预测先行断言                                               | `(?!\.net)`       |
| `(?<=...)`        | 零宽度正回顾后发断言                                               | `(?<=800-)`       |
| `(?<!...)`        | 零宽度负回顾后发断言                                               | `(?<!192\.168\.)` |
| `(?(id/name)Y|N)` | 如果分组提供的id或者name存在，就返回正则表达式的条件匹配Y，否则返回N | `(?x), (? im)`    |

## 需要做进一步解释的正则表达式

在上一部分，我们了解了正则表达式的构成，其中有些需要做进一步的解释。

### 特殊标记嵌入

除了在`re.compile`中传递`flags`参数，我们可以利用`(?iLmsux)`来嵌入特殊标记。以下举几个例子：

```python
>>> re.findall(r'(?i)yes', 'yes? Yes. YES!!')
['yes', 'Yes', 'YES']
```

在这个例子中，通过`(?i)`实现了不区分大小写的正则搜索。

再看一个支持多行并且不区分大小写的正则搜索例子：

```python
>>> re.findall(r'(?im)^th[\w ]+', """
... This line is the first,
... another line,
... that line, it's the best
...""")
['This line is the first', 'that line']
```

### 零宽度正预测先行断言

零宽度正预测先行断言即`(?=...)`。举个例子加以说明。

对正则表达式`(?=\.com)`而言，如果一个字符串后面**跟着**`.com`才做匹配操作，否则不进行。

```python
>>> re.findall(r'(?m)\w+(?=\.com)', """
... https://www.google.com
... https://example.net
... """)
['google']
```

{{< figure src="followed-by-dotcom.png" title="仅匹配后面跟着.com的字符串" numbered="true" >}}

### 零宽度负预测先行断言

零宽度负预测先行断言即`(?!...)`。举个例子加以说明。

对正则表达式`\b(?!un)\w+\b`而言，如果一个单词的开始后面**没有跟着**`un`才做匹配操作，否则不进行。

```python
>>> re.findall(r'\b(?!un)\w+\b', 'unsure sure unity used')
['sure', 'used']
```

{{< figure src="unfollowed-by-un.png" title="仅匹配单词开始后没有跟着un的字符串" numbered="true" >}}

### 零宽度正回顾后发断言

零宽度正回顾后发断言即`(?<=...)`。举个例子以说明。

对于正则表达式`(?<=19)\d{2}\b`而言，如果某个四位数字以19开始，则进行匹配，否则不进行。

```python
>>> re.findall(r'(?<=19)\d{2}\b', '1851 1999 1950 1905 2003')
['99', '50', '05']
```

{{< figure src="preceded-by-19.png" title="仅对以19开始的四位数字进行匹配操作" numbered="true">}}

### 零宽度负回顾后发断言

零宽度负回顾后发断言即`(?<!...)`。举个例子以说明。

对于正则表达式`(?<!19)\d{2}\b`而言，如果某个四位数字不以19开始，则进行匹配，否则不进行。

```python
>>> re.findall(r'(?<=19)\d{2}\b', '1851 1999 1950 1905 2003')
['51', '03']
```
{{< figure src="not-preceded-by-19.png" title="仅对不以19开始的四位数字进行匹配操作" numbered="true">}}

## 链接

* [微软：正则表达式语言-快速参考](https://docs.microsoft.com/zh-cn/dotnet/standard/base-types/regular-expression-language-quick-reference?redirectedfrom=MSDN)
* [脚本之家：正则表达式30分钟入门教程](https://www.jb51.net/tools/zhengze.html)