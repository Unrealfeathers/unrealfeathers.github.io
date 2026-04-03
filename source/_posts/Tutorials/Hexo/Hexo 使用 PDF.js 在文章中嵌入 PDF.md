---
title: Hexo 使用 PDF.js 在文章中嵌入 PDF
date:  2025-04-04 13:59:07

math: false
mermaid: false
category_bar: false

categories:
  - Tutorials

tags:
  - Hexo

excerpt: "使用 PDF.js 在文章中嵌入 PDF"
permalink: /posts/20250404-135907.html
---
## 1. 下载 PDF.js

> [PDF.js - Home](https://mozilla.github.io/pdf.js/)

进入 PDF.js 的官网下载压缩包，将其解压后重命名目录为 `PDF.js`。

## 2. 添加 PDF.js

1. 在 Hexo 根目录的 `source` 目录下，新建 `js` 目录，将 `PDF.js` 放入 `js` 目录；

2. 然后同样在 `source` 目录下，新建 `pdf` 目录，用于放入你的 PDF 文件；

3. 修改配置文件 `_config.yml`，搜索 `skip_render`，修改如下：

```YAML
skip_render: 
  - 'js/PDF.js/**'
```

## 3. 文章中添加 PDF

格式如下：

```HTML
<div>
	<iframe src="/js/PDF.js/web/viewer.html?file=/pdf/example.pdf" width="100%" height="800px"></iframe>
</div>
```

效果如下：

<div>
	<iframe src="/js/PDF.js/web/viewer.html?file=/pdf/高等数学 微积分公式大全.pdf" width="100%" height="800px"></iframe>
</div>
