---
title: Git tag 的使用
date:  2025-06-08 15:27:07

math: false
mermaid: false
category_bar: false

categories:
  - Tutorials

tags:
  - Git

excerpt: "如何使用 git tag 打标签"
permalink: /posts/20250608-152707.html
---
## 1. 标签

在 Git 中，标签主要用于给代码历史中的某一个提交打上标记，通常用于标记重要的里程碑，比如软件的发布版本（例如：V1.0.0）。

Git 的标签分为两种类型：

- 轻量标签：一个指向特定提交的引用，本质上是将提交校验和保存到一个文件中，不包含其他信息；

- 附注标签：存储在 Git 数据库中的一个完整对象。它包含打标签者的名字、电子邮件地址、日期时间，以及一个标签信息，并且可以使用 GPG 进行签名；

## 2. 创建标签

### 2.1 轻量标签

```Bash
# 在当前提交创建标签
git tag V1.0.0

# 在历史提交创建标签
git tag V1.0.0 <commit-hash>
```

### 2.2 附注标签

```Bash
# 在当前提交创建标签
git tag -a V1.0.0 -m "提交消息"

# 在历史提交创建标签
git tag -a V1.0.0 <commit-hash> -m "提交消息"
```

## 3. 查看标签

### 3.1 列出所有标签

```Bash
git tag
```

### 3.2 查找标签

```Bash
git tag -l "V1.0.*"
```

### 3.3 查看标签信息

```Bash
git show V1.0.0
```

## 4. 推送标签

默认情况下，`git push` 命令不会将标签推送到远程服务器。创建完标签后，必须显式地推送到远程仓库。

```Bash
# 推送单个标签
git push origin V1.0.0

# 推送所有标签
git push origin --tags
```

## 5. 删除标签

```Bash
# 删除本地标签
git tag -d V1.0.0

# 删除远程标签
git push origin --delete V1.0.0
# or 推送一个空值到远程标签引用
git push origin :refs/tags/V1.0.0
```

## 6. 检出标签

```Bash
git checkout V1.0.0
```
