---
title: "CyberOrigin 整体系统架构"
description: "CyberOrigin 七层系统架构总览，从硬件设备生产到客户交付的完整数据流"
icon: "layer-group"
---

## CyberOrigin 整体系统架构（七层）

<img src="/assets/cyberorigin_arch_1.svg" alt="CyberOrigin 七层系统架构图" />

## 各层核心设计问题

**L1 硬件生产** — 传感器选型（RGB-D 方案 vs 纯 RGB + 计算深度）、SoC 选择、固件 OTA 更新策略、ODM/OEM 供应链风险管理。

**L2 运营渠道** — 三类渠道的数据密度和质量差异大。消费者侧量大但噪声高；B2B 侧结构化强但场景窄；研究侧质量高但规模有限。渠道策略直接决定数据分布。

**L3 采集层** — 多模态时序同步是核心难题（IMU 高频 vs 视频帧率 vs GPS 低频）。设备端的硬件触发 vs 软件对齐各有取舍。

**L4 传输链路** — 边缘端压缩（有损 vs 无损）对下游 QC 影响大；5G 与 WiFi 混合场景的断点续传状态管理；数据落地加密和合规审计链。

**L5 云端 QC** — 这是整个平台的质量瓶颈，涉及：自动 QC 的误拒率控制、HITL 的成本和规模化、标注体系的版本一致性、以及 PII 检测和脱敏。

**L6 模型与验证** — 世界模型的评测指标设计（尤其是 EgoCentric 场景没有成熟 benchmark）；算法验证如何与真实数据闭环迭代。

**L7 客户交付** — 数据集的格式标准、模型 API 的 latency/throughput SLA、以及客户侧验收的可解释性（如何让客户信任模型质量）。
