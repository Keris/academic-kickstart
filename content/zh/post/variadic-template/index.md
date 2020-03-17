---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "一个例子导向的对变参模版的理解"
subtitle: ""
summary: "如题"
authors: [杜利强]
tags: ["C++", "variadic-template", "变参模版"]
categories: ["C/C++"]
date: 2020-03-17T10:13:02+08:00
lastmod: 2020-03-17T10:13:02+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: "Smart"
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
最近看C++编程语言第四版[^1]，阅读到线程这一章，作者在讲解mutex时给出了一个例子，自己照着写了一个发现跑不通，研究后发现对变参模版理解不够，便写了这篇文章。

首先，我们看看书本上的例子：

[^1]: 英文名为：[The C++ Programming Language, 4th Edition](https://www.amazon.com/dp/0321958322/ref=cm_sw_em_r_mt_dp_U_xP11DbJ5REYKZ). 作者为Bjarne Stroustrup.

```c++
mutex cout_mutex;

template<typename Arg, typename... Args>
void write(Arg a, Args... tail) {
    cout_mutex.lock();
    cout << a;
    write(tail...);
    cout_mutex.unlock();
}
```
这个例子旨在说明deadlock，因为我们在`write`函数中递归调用它，而该函数在进行输出前需要获取互斥变量。为了解除死锁，我们可以使用`recursive_mutex`，这种类型的mutex允许递归地获取。

基于以上想法，我写了一个例子来进行测试：
```c++
#include <iostream>
#include <mutex>

using namespace std;

recursive_mutex cout_mutex;

template<typename Arg, typename... Args>
void write(Arg a, Args... tail) {
    cout_mutex.lock();
    cout << a;
    write(tail...);
    cout_mutex.unlock();
}

int main(int argc, char* argv[]) {
    write("hello,", "world");
    return 0;
}
```

当我尝试编译以上代码却发现报出错误：
```text
$ g++ test.cc -std=c++17
test.cpp:12:6: error: no matching function for call to 'write'
     write(tail...);
     ^~~~~
test.cpp:12:6: note: in instantiation of function template
      specialization 'write<const char *>' requested here
test.cpp:17:6: note: in instantiation of function template
      specialization 'write<const char *, const char *>' requested here
     write("hello,", "world");
     ^
test.cpp:9:7: note: candidate function template not viable: requires at
      least argument 'a', but no arguments were provided
 void write(Arg a, Args... tail) {
      ^
1 error generated.
```
> 在编译时我使用了c++17标准。

可以看到，编译器报告了一个错误，**对'write'的调用没有匹配的函数**，这是为何？

对于以上的实现，`write("hello,", "world")`可以拆分成：

1. 输出`hello,`，此时tail包含一个参数即`world`
2. 输出`world`，此时tail为空，但write函数至少需要一个参数，所以产生以上错误。

下面我们增加一个接受空参数的write函数来验证：

```c++
#include <iostream>
#include <mutex>

using namespace std;

recursive_mutex cout_mutex;

void write() {
    cout << ""; // output nothing
}

template<typename Arg, typename... Args>
void write(Arg a, Args... tail) {
    cout_mutex.lock();
    cout << a;
    write(tail...);
    cout_mutex.unlock();
}


int main(int argc, char* argv[]) {
    write("hello,", "world");
    return 0;
}
```
在进行上述操作后，代码编译通过，运行后输出以下结果：

```text
$ ./a.out
$ hello,world
```