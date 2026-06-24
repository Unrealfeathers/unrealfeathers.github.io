---
title: systemd-timesyncd 设置 NTP 服务器和时区
date:  2026-06-24 18:58:26

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - NTP

excerpt: "使用 systemd-timesyncd 设置 NTP 服务器和时区"
permalink: /posts/20260624-185826.html
---
## 1. 环境

- Ubuntu 22.04 LTS

## 2. 设置时区

执行以下命令，将系统时区更改为 亚洲/上海：

```Bash
sudo timedatectl set-timezone Asia/Shanghai
```

## 3. 设置 NTP 服务器

首先编辑配置文件，打开 `/etc/systemd/timesyncd.conf` 文件：

```Bash
sudo vim /etc/systemd/timesyncd.conf
```

取消 `NTP=` 和 `FallbackNTP=` 行的注释，并填入 NTP 服务器地址：

```Ini
[Time]
NTP=ntp1.ntsc.ac.cn ntp1.aliyun.com
FallbackNTP=ntp1.tencent.com ntp.ubuntu.com
#RootDistanceMaxSec=5
#PollIntervalMinSec=32
#PollIntervalMaxSec=2048
```

然后重启服务，应用配置：

```Bash
# 开启 NTP 同步服务
sudo timedatectl set-ntp true

# 重启 systemd-timesyncd 服务
sudo systemctl restart systemd-timesyncd
```

## 4. 查看状态

1. 运行以下命令查看同步状态：

```Bash
timedatectl timesync-status
```

{% note info %}
在输出中，查看 `Server:` 字段是否指向了设置的 NTP 服务器，并检查 `Packet count:` 是否有数据包交互。
{% endnote %}

2. 运行以下命令查看当前时间：

```Bash
timedatectl
```

输出：

```Text
               Local time: Wed 2026-06-24 20:00:00 CST
           Universal time: Wed 2026-06-24 12:00:00 UTC
                 RTC time: Wed 2026-06-24 12:00:00
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

重点查看以下字段：

- `System clock synchronized: yes`：系统时钟已同步

- `NTP service: active`：NTP 服务已激活
