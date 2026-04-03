---
title: Hexo V8.1.0 & Fluid V1.9.8 部署指南
date:  2026-02-08 21:14:52

math: false
mermaid: false
category_bar: false

categories:
  - Tutorials

tags:
  - Hexo
  - Fluid

excerpt: "部署基于 Hexo 的 Blog 工程于云服务器"
permalink: /posts/20260208-211452.html
---
## 1. Hexo V8.1.0

> [Hexo](https://hexo.io/)

默认已安装 `Git`，此处不在叙述。本教程使用云服务器部署工程，可以随时随地大小写文章。所有操作都基于 Ubuntu Server 24.04.4 LTS 系统。

### 1.1 Node.js

使用 `NodeSource` 安装：

```Bash
# 更新并安装必要工具
sudo apt update
sudo apt install -y curl

# 下载并执行 NodeSource 设置脚本
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -

# 安装 Node.js
sudo apt install -y nodejs

# 验证安装
node -v
npm -v
```

### 1.2 Hexo

```Bash
# 安装
sudo npm install -g hexo-cli

# 初始化，<folder> 为你的博客目录。
hexo init <folder>
cd <folder>
npm install

# 预览
hexo server
```

## 2. Fluid

> [开始使用 | Hexo Fluid 用户手册](https://fluid-dev.github.io/hexo-fluid-docs/start/)

### 2.1 安装

进入博客目录执行命令：

```Bash
npm install --save hexo-theme-fluid
```

### 2.2 `_config.yml`  配置

#### 2.2.1 修改主题

打开博客根目录下的 `_config.yml`（注意不是主题文件夹里的）：

```YAML
# 指定语言
language: zh-CN

# 指定主题
theme: fluid
theme_config:
  # 主题配置文件
  theme_config_file: _config.fluid.yml
```

#### 2.2.2 主题配置文件

Hexo 8.x 推荐使用“覆盖配置”的方式，而不是直接去修改`node_modules`里的文件。

1. 在博客根目录下新建一个文件：`_config.fluid.yml`；

2. 将主题的 [_config.yml](https://github.com/fluid-dev/hexo-theme-fluid/blob/master/_config.yml) 内容复制过去；

3. 后续所有的个性化配置都在这个 `_config.fluid.yml` 中修改；

### 2.3 删除默认主题

```bash
npm uninstall --save hexo-theme-landscape
```

### 2.4 预览

```bash
hexo server
```
