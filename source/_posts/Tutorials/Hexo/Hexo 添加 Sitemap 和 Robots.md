---
title: Hexo 添加 Sitemap 和 Robots
date:  2025-04-11 17:22:55

math: false
mermaid: false
category_bar: false

categories:
  - Tutorials

tags:
  - Hexo

excerpt: "添加 Sitemap 和 Robots 来进行 SEO 优化"
permalink: /posts/20250411-172255.html
---
## 1. 添加 Sitemap

### 1.1 安装插件

打开终端或者命令行，进入你的 Hexo 博客根目录，运行以下命令安装插件：

```Bash
# 安装通用 Sitemap 生成器
npm install hexo-generator-sitemap --save
```

### 1.2 修改配置文件

打开 Hexo 根目录下的 `_config.yml` 文件，首先确保网站的网址正确，然后添加插件配置：

```YAML
# Url 配置
url: https://flowerdown.org

# Sitemap 配置
sitemap:
  path: sitemap.xml
```

{% note info %}
记得修改网址为你的域名；
{% endnote %}

## 2. 添加 Robots.txt

`robots.txt` 是告诉搜索引擎爬虫哪些页面可以抓取，哪些不能抓取的文件。进入你 Hexo 项目的 `source` 文件夹（注意是 `source` 文件夹，不是根目录），在里面新建一个名为 `robots.txt` 的纯文本文件，然后输入内容：

```Text
User-agent: *
Allow: /
Allow: /archives/
Allow: /categories/
Allow: /tags/
Disallow: /js/
Disallow: /css/
Disallow: /fonts/
Disallow: /vendors/

Sitemap: https://flowerdown.org/sitemap.xml
```

{% note info %}
记得修改网址为你的域名；
{% endnote %}
