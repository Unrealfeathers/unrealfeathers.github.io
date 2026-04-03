---
title: Fluid 去除主题默认图片
date:  2026-02-01 18:02:54

math: false
mermaid: false
category_bar: false

categories:
  - Tutorials

tags:
  - Hexo
  - Fluid

excerpt: "去除主题默认图片，提高页面加载速度"
permalink: /posts/20260201-180254.html
---
## 编辑 `_config.yml` 文件

在 `_config.yml` 文件中搜索 `ignore`，添加如下内容，这样就可以忽略掉主题内部的默认图片，此修改在 `Hexo` 和 `Fluid` 更新后同样生效。

```YAML
ignore:
  - "**/node_modules/hexo-theme-fluid/source/img/*"
```
