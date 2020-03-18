---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "随机数生成"
subtitle: "C/C++中的随机数生成"
summary: "在C/C++中如何生成真随机数"
authors: [杜利强]
tags: ["C/C++", "PRNGs", "random-number-generator", "随机数生成"]
categories: ["C/C++"]
date: 2020-03-18T11:45:54+08:00
lastmod: 2020-03-18T11:45:54+08:00
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
最近参加了一家公司的面试，其中一个问题是：“如何在C语言中生成一个真随机数？” 我当时想到的答案是使用`rand()`，但`rand()`返回的其实是一个伪随机数，值的范围是`[0, RAND_MAX]`。其中，`RAND_MAX`跟具体的实现有关，也就是说在不同的实现中可能有不同的值。

以上便是这篇博客的背景，也可以看出自己对随机数生成（RNG）掌握的也不全面，所以试图通过此文弥补一下短板。

## 随机数的应用
随机数的应用有很多方面，包括赌博，统计采样，计算机仿真，加密等。

## 真随机数与伪随机数
简单来讲，真随机数无法预测，而伪随机数在知道种子值后便能复现。我们先来了解一下生成随机数的两种方法。

在第一种方法中，我们测量某个被认为是随机的物理现象，然后补偿测量过程中可能的偏差。这种随机源包括空气噪声，热噪声以及其他外部的电磁和量子现象。要产生一个随机数，我们需要获取足够的墒（entropy），但获取速率取决于被测量的物理现象。因此，这种产生随机数的方法是阻塞的（blocking）的，速率受限的。举个例子，在大部分Linux发行版中，伪随机设备文件`/dev/random`就会阻塞直到从环境获取到足够的墒。由于这种阻塞行为，从`/dev/random`大批量读以便用随机位填充磁盘经常会很慢，就是因为墒源是阻塞型的。

在第二种方法中，我们使用计算的方法来生成明显随机值组成的长序列，但实际上结果完全由一个较短的初始值确定，这个初始值也叫种子值。因此，整个看似随机的序列在知道种子值的情况下能够被复现。相比第一种方法，第二种方法是非阻塞（non-blocking）的，所以其生成速率不受外部事件的影响。

在实际应用中，一些系统采取混合的办法，即在可用的时候采用自然源提供的随机性，否则使用周期性重新设置种子、软件基于的密码安全的伪随机数生成器（CSPRNGs）。

在这里小结一下：真随机数依赖于收集自然发生的墒，有了足够的墒方可生成随机数，因此是阻塞的，速率有限的；伪随机数使用计算方法产生，在知道种子值后能够复现，非阻塞，速率不受限。

## C语言中的随机数生成
在C语言中，标准库`stdlib.h`提供了两个函数`int rand(void)`和`void srand(unsigned)`来帮助我们生成伪随机数。

`rand()`返回一个位于区间`[0, RAND_MAX]`的整数，`RAND_MAX`在不同的实现中可能不同，但被保证至少是`32767`。在调用`rand()`前，如果我们不设置种子，系统默认我们设置种子为1，即调用了`srand(1)`，在这种情况下，我们每次运行将会产生同样的值，下面看一个例子：

```c++
#include <stdio.h>
#include <stdlib.h>

int main() {
    int num = rand();
    printf("Random number: %d\n", num);
    return 0;
}
```
我们编译以上代码并运行5次：

```
$ gcc gen_rand.c && for i in `seq 1 5`;do ./a.out;done
Random number: 16807
Random number: 16807
Random number: 16807
Random number: 16807
Random number: 16807
```
可以看到，程序在每次运行时均生成了同样的值，这是因为种子值均为1。

接下来，我们再看看手动设置种子的情况：
```c++
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main() {
    srand((unsigned)time(0)); // use current time as seed
    int num = rand();
    printf("Random number: %d\n", num);
    return 0;
}
```
这一次，我们将种子设为当前时间。同样地，我们编译并运行5次：
```
$ gcc gen_rand1.c && for i in `seq 1 5`;do ./a.out;done
Random number: 7890298
Random number: 7890298
Random number: 7890298
Random number: 7890298
Random number: 7890298
```
结果看似跟前面没有不同，因为5次调用依然生成了同样的值。其实，这个差别很细微，我们将种子设为当前时间，但只有在重新编译时才能有不同的种子值。因此，两次编译之间生成的值应该不同：
```
$ gcc gen_rand1.c && ./a.out
Random number: 12713907
$ gcc gen_rand1.c && ./a.out
Random number: 12747521
```

## C++中的随机数生成
在C++中，我们仍然可以使用`rand()`和`srand()`来生成随机数，但这已经不是标准推荐的做法，相反我们应该使用`random`库提供的工具类来生成随机数。

下面给出C++版本的代码，运行结果跟C语言别无二致，不再赘述。
```c++
#include <iostream>
#include <cstdlib>
#include <ctime>

using namespace std;

int main() {
    // srand((unsigned)time(nullptr)); // use current time as seed
    int r = rand();
    cout << "Random number between 0 and " << RAND_MAX << " :";
    cout << r << '\n';
    return 0;
}
```

接下来，我们使用`random`库提供的工具来生成随机数。`random`库提供了两种类型的类：

- 均匀分布随机位生成器（URBGs），既包括伪随机数生成和真随机数生成（如果可用）
- 随机数分布

详细的信息请参见 https://en.cppreference.com/w/cpp/numeric/random。

下面给出一个使用随机数引擎生成随机数的例子：
```c++
#include <iostream>
#include <random>

using namespace std;

int main() {
    random_device rd;
    default_random_engine e1(rd());
    cout << e1() << endl;
    return 0;
}
```
我们编译以上代码，并运行5次：

```
$ g++ main.cc -std=c++17 && for i in `seq 1 5`;do ./a.out;done
17429349
872858569
1540494345
40333012
95212997
```
从这里可以看到，我们编译代码一次，多次运行，每次会产生不同的值，序列变得不可预测。

此外，我们可以将生成的随机数转换到某个闭区间的均匀分布或者正态分布，下面是一个均匀分布的例子：

```c++
#include <iostream>
#include <iomanip>
#include <string>
#include <random>
#include <map>

using namespace std;

int main() {
    random_device rd;
    default_random_engine e1(rd());
    uniform_int_distribution<int> dis(1, 6);
    cout << dis.min() << endl;
    cout << dis.max() << endl;
    map<int, int> hist;
    for (int i = 0; i != 10000; i++) {
        hist[dis(e1)]++;
    }
    for (auto[num, count] : hist) {
        cout << num << ' ' << string(count / 200, '*') << '\n';
    }
    return 0;
}
```
编译以上代码并运行，结果如下：
```
$ g++ -std=c++17 main.cc && ./a.out
1
6
1 ********
2 ********
3 *******
4 ********
5 ********
6 ********
```
我们来分析一下。首先，我们输出均匀分布可能生成的最小值和最大值，由于我们考虑的是闭区间`[1, 6]`上的均匀分布，显然最小值和最大值分别为1和6。然后我们将产生的随机数转换到区间上的某个值，并统计出现次数。最后，我们绘制直方图，从结果来看，每个值出现的次数基本一样多（我们进行了10000次实验！）。