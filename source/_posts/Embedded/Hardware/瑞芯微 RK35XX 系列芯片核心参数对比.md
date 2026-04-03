---
title: 瑞芯微 RK35XX 系列芯片核心参数对比
date:  2026-03-07 19:23:53

math: false
mermaid: false
category_bar: false

categories:
  - Embedded System

tags:
  - 瑞芯微

excerpt: "赞美 AI"
permalink: /posts/20460307-192353.html
---

| **芯片型号** | **CPU 架构** | **GPU** | **NPU 算力** | **视频解码 (Max)** | **视频编码 (Max)** | **内存支持** | **市场定位与典型应用** |
| ------------ | ------------ | ------- | ------------ | ------------------ | ------------------ | ----------- | ---------------------- |
| **RK3588** | **8核**<br><br>4×Cortex-A76 (2.4GHz)<br><br>4×Cortex-A55 (1.8GHz) | Mali-G610 MC4 | **6 TOPS** | 8K@60fps | 8K@30fps | 64-bit LPDDR4/4x/5 | **旗舰级**：高端 ARM PC、边缘计算、8K 多媒体显示、复杂 AI 视觉分析。 |
| **RK3576** | **8核**<br><br>4×Cortex-A72 (2.2GHz)<br><br>4×Cortex-A53 (1.8GHz)<br><br>+ Cortex-M0 | Mali-G52 MC3 | **6 TOPS** | 8K@30fps /<br>4K@120fps | 4K@30fps | 32-bit LPDDR4/4x/5 | **次旗舰 AIoT**：高性能机器视觉、工业平板、商业显示、高端智能家居。性价比高于 RK3588。 |
| **RK3568** | **4核**<br><br>Cortex-A55 (2.0GHz) | Mali-G52 2EE | **1 TOPS** | 4K@60fps | 1080p@60fps | 32-bit DDR3/DDR4<br><br>LPDDR4/4x | **中高端全能**：工控主板、NVR（网络录像机）、NAS 存储、智能网关。接口极为丰富（多网口、PCIe、SATA）。 |
| **RK3568J** | **4核**<br><br>Cortex-A55 (1.4~1.8GHz) | Mali-G52 2EE | **1 TOPS** | 4K@60fps | 1080p@60fps | 32-bit DDR3/DDR4<br><br>LPDDR4/4x (支持ECC) | **工业级定制版**：支持宽温 (-40℃ ~ 85℃) 和全链路 ECC，专为严苛的工业自动化、车载终端设计。 |
| **RK3566** | **4核**<br><br>Cortex-A55 (1.8GHz) | Mali-G52 2EE | **1 TOPS** | 4K@60fps | 1080p@60fps | 32-bit DDR3/DDR4<br><br>LPDDR4/4x | **中端消费级**：核心性能与 RK3568 类似，但裁剪了部分外围接口，主要用于平板电脑、电视盒子、电子墨水屏等消费类电子。 |
| **RK3562** | **4核**<br><br>Cortex-A53 (1.4GHz) | Mali-G52 2EE | **1 TOPS** | 4K@30fps | 1080p@60fps | 32-bit LPDDR4/4x | **入门级 AIoT**：主打高性价比，适合基础工业控制、扫地机器人、智能音频及考勤门禁设备。 |
| **RK3506** | **3核**<br><br>Cortex-A7 (1.5GHz)<br><br>+ Cortex-M0 | 无3D GPU<br><br>(仅2D图形引擎) | 无 | 720p@30fps<br>(软件解码) | 无 / MJPEG | 16-bit DDR3/DDR3L<br><br>(通常内置 256/512MB) | **低功耗入门级**：极致性价比，用于带简单屏幕的工业 HMI（人机界面）、智能语音交互、低功耗物联网网关。 |

## 选型指南总结：

1. **追求极致性能与 8K 视频处理**：首选 **RK3588**；如果预算有限但需要强大的 AI 算力（6 TOPS），**RK3576** 是目前最具性价比的平替方案；

2. **需要大量接口（PCIe/SATA/双千兆网口）做软路由、NAS 或工控**：**RK3568** 是不二之选；如果应用环境恶劣（如极寒、高温），则选用工业级的 **RK3568J**；

3. **消费级产品（平板、机顶盒）**：选择 **RK3566**，它具有 RK3568 同等的音视频能力，但成本更低；

4. **低功耗与极低成本控制**：**RK3562**（带轻量 AI）或 **RK3506**（纯基础控制与语音交互、常用于替代传统的全志或百问网等低端 Linux 方案）最为合适；
