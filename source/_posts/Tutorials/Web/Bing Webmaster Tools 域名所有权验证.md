---
title: Bing Webmaster Tools 域名所有权验证
date:  2026-03-30 18:17:05

math: false
mermaid: false
category_bar: false

categories:
  - Tutorials

tags:
  - Web

excerpt: "验证域名所有权"
permalink: /posts/20260330-181705.html
---
## 1. 获取 Bing 提供的 CNAME 信息

在 Bing Webmaster Tools 的验证页面，选择验证方式为 CNAME 记录。Bing 此时会为你生成两段信息：

- 名称/主机 (Name/Host)：一串较长的随机字符；

- 值/目标 (Value/Target)：通常固定为 `verify.bing.com`；

## 2. 在 Cloudflare 中添加记录

1. 登录 Cloudflare 仪表板；

2. 进入到需要验证的域名的管理界面；

3. 在左侧导航栏中，点击“DNS -> 记录 (Records)”；

4. 点击蓝色的“添加记录 (Add record)”按钮；

按照以下说明填写记录表单：

1. 类型 (Type)：在下拉菜单中选择 CNAME；

2. 名称 (Name)：粘贴在 Bing 复制的那串长字符；

3. 目标 (Target)：输入 `verify.bing.com`；

4. 代理状态 (Proxy status)：点击切换开关，将其从橙色的“已代理 (Proxied)”状态，切换为灰色的“仅限 DNS (DNS only)”；

5. TTL：保持默认的“自动 (Auto)”即可；

6. 最后点击右下角的“保存 (Save)”；

## 3. 完成验证

切换回 Bing Webmaster Tools 的网页，点击底部的“验证 (Verify)”按钮。如果设置无误，Bing 会立即提示域名所有权验证成功。

{% note warning %}
验证成功后，不要删除 Cloudflare 中的这条 CNAME 记录。Bing 会定期检查验证状态，如果记录丢失，可能会导致你的网站从站长工具中掉签。
{% endnote %}
