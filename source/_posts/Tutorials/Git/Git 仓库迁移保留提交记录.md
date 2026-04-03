---
title: Git 仓库迁移保留提交记录
date:  2026-02-26 18:50:51

math: false
mermaid: false
category_bar: false

categories:
  - Tutorials

tags:
  - Git

excerpt: "将当前分支的代码迁移到另一个仓库的分支"
permalink: /posts/20260226-185051.html
---
## 1. 准备工作

首先，进入**目标仓库**本地目录。如果本地没有，请先克隆它：

```Bash
git clone <目标仓库地址>
cd <目标仓库目录>
```

## 2. 添加源仓库作为远程地址

为了让目标仓库知道源仓库的存在。将源仓库添加为一个名为 `temp_origin` 的远程分支：

```Bash
git remote add temp_origin <源仓库地址>
```

## 3. 获取源仓库数据

从源仓库拉取所有的数据和分支信息到本地，但此时并不会合并任何文件:

```Bash
git fetch temp_origin
```

## 4. 创建并切换到新分支

在目标仓库中创建一个新分支（例如：`migrated-content`），用来接收源仓库的代码：

```Bash
# 基于源仓库的 master/main 分支创建新分支
git checkout -b migrated-content temp_origin/master
```

{% note warning %}
如果源仓库的主分支叫 `main`，请将 `master` 改为 `main`
{% endnote %}

## 5. （可选）整理文件结构

如果你希望源仓库的所有内容都存放在目标仓库的一个**特定子目录**下，而不是直接覆盖根目录，请执行以下操作：

```Bash
# 创建子目录并将所有文件移动进去
mkdir source_repo_folder
git ls-tree -z -r --name-only HEAD | xargs -0 -I {} mv {} source_repo_folder/

# 提交这个移动操作
git add .
git commit -m "Move source files to sub-directory"
```

## 6. 合并到目标分支

现在，切换回目标仓库想要合并源仓库进去的分支（比如 `main` 或 `developer`），然后合并刚才处理好的分支：

```Bash
git checkout main

# 合并分支，使用 --allow-unrelated-histories 是关键
# 因为这两个仓库没有共同的提交历史
git merge migrated-content --allow-unrelated-histories
```

## 7. 推送与清理

最后，将合并后的内容推送到目标仓库的远程端，并删除临时配置：

```Bash
# 推送
git push origin main

# 删除临时远程连接
git remote remove temp_origin

# 删除本地临时分支
git branch -d migrated-content
```
