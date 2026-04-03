---
title: Git 恢复最新提交
date:  2025-06-07 13:23:02

math: false
mermaid: false
category_bar: false

categories:
  - Tutorials

tags:
  - Git

excerpt: "切换到特定提交之后，重新恢复到最新提交"
permalink: /posts/20250607-132302.html
---
## 1. 切换

```Bash
# 切换到特定提交
git checkout <commit-hash>
```

## 2. 恢复

使用 `git checkout` 切换到特定提交后，仓库通常处于一种被称为“头指针分离”的状态。在这种状态下，你并不在任何分支上，而是在一个特定的提交上。所以要恢复到最新提交，最简单的方法就是切换回原来的分支名。

```Bash
# 查看所有分支
git branch -a

# 切换到主分支
git switch main
```

如果在历史提交的基础上进行代码修改并提交了，直接切换分支会导致新提交变成孤儿提交。如果需要保留修改，可参考如下步骤：

```Bash
# 创建临时分支
git checkout -b temp-branch

# 切换到主分支
git checkout main

#合并修改
git merge temp-branch
```

如果很不幸出现了刚刚说到的孤儿提交，勿慌，还有挽回的机会。

```Bash
# 查看日志，找到孤儿提交的哈希值
git reflog

# 找回提交
git branch recover-branch <commit-hash>
```
