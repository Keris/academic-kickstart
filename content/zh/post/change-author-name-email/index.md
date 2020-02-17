---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "更改git提交中的作者和邮件信息"
subtitle: ""
summary: ""
authors: []
tags: ["git", "change-author"]
categories: []
date: 2019-11-07T17:47:23+08:00
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
在git的日常使用中更改提交的作者名和邮箱是一个十分常见的操作。例如，当你克隆了一个项目，如果你没有进行任何设置，此时提交的作者和邮件将使用全局选项，这可能不是你所要的。

<!--more-->

通过本文你将了解：

- 如何配置git使用的提交者用户名和邮箱
- 如何更改最近一次提交的作者名和邮件
- 批量更改提交的作者名和邮件

## 配置提交者用户名和邮箱

你有两种方式进行设置，一是全局配置，二是为每个repo单独配置。

### 全局地更改提交者用户名和邮箱
使用`git config`并`--global`选项进行全局设置，如下所示：
```text
$ git config --global user.name "Du Liqiang"
$ git config --global user.email "dlq137@gmail.com"
```
设置完毕后，后续的提交将使用以上提供的信息。

### 为某个repo单独配置
全局选项可能并不适用于某个repo，此时就需要单独设置，使用`git config`但省略`--global`选项，如下：
```text
$ git config user.name "Du Liqiang"
$ git config user.email "dlq137@gmail.com"
```
这里的设置将覆盖全局选项，并且只应用于当前的repo。

## 更改最近一次提交的作者用户名和邮箱
如果你刚做了一次提交，发现用户名和邮箱并不是所要的，你可以使用`--amend`选项重新提交：
```text
$ git commit --amend --author="Du Liqiang <dlq137@gmail.com>"
```
## 更改多次提交的作者用户名和邮箱
这个时候我们需要借助强大的`rebase`命令，首先我们找到上一次“好”的提交[^1]，并假设其提交hash为`0ad14fa5`，执行：
[^1]: 该次提交之前的提交具有正确的用户名和邮箱，其后的需要进行更改。
```text
$ git rebase -i -p 0ad14fa5
```
此时我们会进入一个编辑器，将那些需要编辑的提交全标记为`edit`，接下来git会指导你完成每次提交的编辑，你需要做的就是执行：
```
$ git commit --amend --author="Du Liqiang <dlq137@gmail.com> --no-edit
$ git rebase --continue
```

## 使用git filter-branch批量更改
除了以上交互式的更改方法，另一种方法是借助git的`filter-branch`命令，其允许你使用一个script批量处理大量的提交。

如下命令筛选提交邮箱为WRONG_EMAIL的提交，并将其用户名和邮箱分别设置为NEW_NAME和NEW_EMAIL对应的值。
```text
$ git filter-branch --env-filter '
WRONG_EMAIL="wrong@example.com"
NEW_NAME="New Name Value"
NEW_EMAIL="correct@example.com"

if [ "$GIT_COMMITTER_EMAIL" = "$WRONG_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$NEW_NAME"
    export GIT_COMMITTER_EMAIL="$NEW_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$WRONG_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$NEW_NAME"
    export GIT_AUTHOR_EMAIL="$NEW_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

## 参考
- https://www.git-tower.com/learn/git/faq/change-author-name-email
