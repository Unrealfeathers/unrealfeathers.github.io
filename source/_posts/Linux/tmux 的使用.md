---
title: tmux 的使用
date:  2026-05-24 19:35:55

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - tmux

excerpt: "如何使用 tmux 创建后台进程"
permalink: /posts/20260524-193555.html
---
## 1. 基础会话管理

在终端中直接输入以下命令来管理 tmux 会话：

```Bash
# 新建匿名会话
tmux

# 新建命名会话
tmux new -s <会话名>

# 查看当前运行的所有会话
tmux ls

# 重新连接到已有会话
tmux attach -t <会话名>
# or
tmux attach -t <会话编号>

# 退出当前会话并使其在后台运行
tmux detach

# 彻底结束并在内部退出当前会话
exit

# 彻底关闭指定会话
tmux kill-session -t <会话名>
# or
tmux kill-session -t <会话编号>

# 切换到另一个会话
tmux switch -t <会话名>
# or
tmux switch -t <会话编号>

# 重命名会话
tmux rename-session -t <会话名 | 会话编号> <新会话名>
```

## 2. 前缀键

进入了 tmux 之后，绝大多数的快捷键操作都需要先按一个前缀键，然后再按具体的命令键。

- 默认前缀键是：`Ctrl + b`；

## 3. 常用快捷键

以下操作全都在进入 tmux 之后进行：

### 3.1 帮助

查看帮助信息：前缀键 + ?

使用 `ESC` 或者 `q` 退出。

### 3.2 退出和挂起

挂起当前会话，使其在后台运行：前缀键 + d

### 3.3 列出所有会话

列出所有会话：前缀键 + s

### 3.4 重命名

重命名当前会话：前缀键 + $
