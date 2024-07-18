---
title: git checkout检出文件
date: 2024-07-18 22:01:04
categories:
    - 运维
tags: 
    - git
---

# 概述

git checkout的作用通常有两个：

- 切换到指定分支或者commit
- 从指定分支或commit中恢复文件到当前工作目录

第一个作用大家都很清楚，通过 `git checkout <branch>|<commit id>` 很轻松能做到。本文重点记录第二个作用，也就是检出文件的作用。

# 背景

我学习项目的时候通常喜欢在README.md中记录笔记。然后现在我希望将当前项目的版本回退到前一个版本，但是同时我又希望只保留当前版本的 README.md中的新增笔记内容。

如果直接使用 `git revert <commit id>`  那么会直接恢复到前一个版本，但是随之而来的，我的新增的笔记内容会消失。使用 `git reset` 也是一样。下方git树图示将展示我希望git树的变化:

```shell
# 当前git树：
a (HEAD -> main)
|
b
|
c

# 希望的git树：(回退版本，但是保留特定文件)
b (HEAD -> main)
|
c
```

# 解决方案

通过 `git checkout` 检出文件的功能，可以实现此目的。

```shell
git checkout <branch> -- <file> ...
```

新建一个临时分支，这个分支保留了主分支最新的提交。然后在主分支执行回滚操作，回滚到指定的版本。然后通过`git checkout` 的文件检出功能，将之前的临时分支中最新的文件内容检出到当前分支。就实现了 ”版本回退但是保留特定文件最新状态“ 的目的。



具体流程如下：

## 流程

1. 新建一个临时分支，该分支会保存当前分支的所有最新提交
   
   ```shell
   git checkout -b temp-branch
   ```
   
   此时，我们的git树会像这样：
   
   ```shell
   # 当前git树：
   a temp-branch(HEAD -> temp-branch)
   |/
   b
   |
   c
   ```

2. 此时temp-branch相当于main分支的备份。我们现在可以放心的回滚main分支的提交了。回滚提交到指定的版本：
   
   ```shell
   # 切换到main分支的最新提交（a提交）
   git checkout main
   
   # 重置HEAD到b
   git reset --hard b 
   ```
   
   此时，我们的git树会像这样：
   
   ```shell
   # 当前git树：
   a temp-branch
   |/
   b (HEAD -> main)
   |
   c
   ```

3. 此时README文件中的内容也一同回滚到b的版本了，但是别担心，我们可以使用`git checkout` 指令来恢复文件。
   
   还记得`temp-branch`分支吗？他是main分支最新提交的备份，该分支中保存了a提交中的最新内容。因此，我们使用`git checkout` 来从`temp-branch`分支中恢复内容：
   
   ```shell
   # 将temp-branch分支中的README.md检出到当前版本中
   git checkout temp-branch -- README.md
   ```
   
   假如现在我改变了想法，我除了想保留README.md，还想保留之前版本的所有yml文件，那么可以这样写：
   
   ```shell
   git checkout temp-branch -- README.md $(find ./ -name '*.yml')
   ```
   
   完成这一切后，删除这个临时分支（它已经发挥了它的作用）。我们的目的达到了。
   
   ```shell
   git checkout -d temp-branch
   ```


