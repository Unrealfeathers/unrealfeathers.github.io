---
title: Hexo 更新软件包
date:  2026-02-07 16:12:07

math: false
mermaid: false
category_bar: false

categories:
  - Tutorials

tags:
  - Hexo

excerpt: "如何更新软件包版本"
permalink: /posts/20260207-161207.html
---
## 1. 常规安全更新

```Bash
# 检查过时包
npm outdated

# 更新到兼容版本
npm update
```

## 2. 深度更新

如果需要将 Hexo 或者插件升级到最新的版本，建议使用 `npm-check-updates`。

{% note danger %}
更新前，请务必进行一次 Git 提交，或者备份 `_config.yml` 和你的主题配置文件！！！
{% endnote %}

### 2.1 安装

```Bash
sudo npm install -g npm-check-updates
```

### 2.2 检查版本

注意要在 Hexo 根目录下运行：

```Bash
ncu
```

在确认所有软件包的更新没有问题后，运行以下命令修改 `package.json`：

```Bash
ncu -u
```

### 2.3 软件包更新

最后执行软件包安装：

```Bash
npm install
```
