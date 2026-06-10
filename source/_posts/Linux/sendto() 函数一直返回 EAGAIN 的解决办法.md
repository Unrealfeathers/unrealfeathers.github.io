---
title: sendto() 函数一直返回 EAGAIN 的解决办法
date:  2026-05-31 19:06:31

math: false
mermaid: false
category_bar: false

categories:
  - Linux

tags:
  - Socket
  - UDP
  - C

excerpt: "Linux SNDBUF 缓冲区写满"
permalink: /posts/20260531-190631.html
---
## 1. 环境

- Ubuntu Desktop 24.04.4 LTS

## 2. 报错信息

在调试程序中，我发现有时日志输出 UDP 数据发送完成，但是对端设备并未收到数据包。于是排查中，发现 `sendto()` 函数在非阻塞模式下，返回值为 -1，随即打印出 `errno` 为 **11(EAGAIN)** 。

之后添加了发送重试的处理，发现程序启动时仍会有大量报错，多个不重复的数据包发送重试三次仍然失败，`errno` 为 **11(EAGAIN)** 。之后尝试了多种办法，延长重试等待时间、增加重试次数、更换端口、 **程序内** 增加 SNDBUF 缓冲区长度......

最后终于发现是系统内核分配的 SNDBUF 缓冲区写满导致的。

## 3. 报错原因

如上所述，我已经在**程序内**增加了 SNDBUF 缓冲区长度，为什么缓冲区还会写满呢？

设置 SNDBUF 缓冲区长度：

```C
    /* 设置发送缓冲区为 2MB */
    int send_buf_size = 2 * 1024 * 1024; 
    setsockopt(sock_fd, SOL_SOCKET, SO_SNDBUF, &send_buf_size, sizeof(send_buf_size));
```

这时需要在命令行输入以下命令：

```Bash
sudo sysctl net.core.wmem_max net.core.wmem_default
```

输出：

```Text
net.core.wmem_max = 212992
net.core.wmem_default = 212992
```

这样就真相大白了：在设置前，系统会拿程序中的设定值去对比 **系统全局上限 `net.core.wmem_max` (208KB)** 。如果设定值超限，内核会直接忽略申请 2MB 缓冲区的请求，强行把这个 Socket 的发送缓冲区限制在上限值内。

那么你可能疑惑了（如果你做了返回值检查的话），为什么 `setsockopt()` 返回了 **0** 。这就是 Linux 网络编程中非常著名的“静默截断（Silent Truncation）”陷阱，系统在截断你的申请之后，并未抛出警告或异常。

为了验证这个问题，修改程序如下：

```C
    /* 设置发送缓冲区为 2MB */
    int send_buf_size = 2 * 1024 * 1024;
    int ret = setsockopt(sock_fd, SOL_SOCKET, SO_SNDBUF, &send_buf_size, sizeof(send_buf_size));
    if (ret == 0) {
        int actual_buf_size = 0;
        socklen_t optlen = sizeof(actual_buf_size);
        getsockopt(sock_fd, SOL_SOCKET, SO_SNDBUF, &actual_buf_size, &optlen);

        printf("Requested SNDBUF: %d, Actual SNDBUF: %d, ret %d;\r\n", send_buf_size, actual_buf_size, ret);
    }
```

输出：

```Text
Requested SNDBUF: 2097152, Actual SNDBUF: 425984, ret 0;
```

{% note info %}
注：实际读出来的值通常是 `wmem_max` 的两倍（212992 * 2 = 425984 字节）。这是因为 Linux 内核会自动把设置的值翻倍，额外的一半用来存放 `sk_buff` 内部的数据结构开销，实际有效 `payload` 空间依然是被 `wmem_max` 死死卡住的；
{% endnote %}

## 4. 解决办法

知道了原因就很好解决了，直接把上限值设大就可以了，命令如下：

### 4.1 临时修改

```Bash
# wmem_max 8MB
sudo sysctl -w net.core.wmem_max=8388608

# wmem_default 8MB
sudo sysctl -w net.core.wmem_default=8388608
```

### 4.2 永久修改

```Bash
sudo vim /etc/sysctl.conf
```

在文件末尾追加：

```Text
net.core.wmem_max = 8388608
net.core.wmem_default = 8388608
```

保存退出后，执行以下命令使配置加载：

```Bash
sudo sysctl -p
```

{% note info %}
注：调大缓冲区只能应对突发流量的排队问题。如果发送速率长期大于网卡的物理承载能力，建议在代码逻辑中加入令牌桶等限速机制，或者在捕获到 `EAGAIN` 时短暂 `usleep()` 让内核有时间清空队列；
{% endnote %}

然后去跑一个压测试试叭，如果还是不行，你可能需要另请高明了。