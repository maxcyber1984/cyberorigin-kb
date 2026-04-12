---
title: "公司概览"
description: "CyberOrigin 是谁、做什么、核心技术判断以及当前阶段定位"
icon: "building"
---

## 我们是谁

CyberOrigin 是一家专注于**具身智能数据基础设施**的公司。我们为机器人和具身智能系统提供高质量的 EgoCentric（第一人称视角）训练数据，覆盖从现场采集、标注质检到结构化交付的完整链路。

团队规模 20-40 人，目前处于中期阶段，以深圳为主要运营基地。

---

## 核心技术判断

我们的核心赌注建立在三个判断上：

**1. 数据是具身智能的瓶颈**
当前机器人策略训练和世界模型研究面临的根本障碍不是算法，而是高质量、多样化的真实世界交互数据的匮乏。

**2. EgoCentric 数据具有独特价值**
第一人称视角数据是训练机器人"理解并执行人类动作"的最自然形式。相比第三人称数据，EgoCentric 数据在手部动作精度、空间关系感知和任务步骤建模上具有系统性优势。

**3. 完整链路工程化是护城河**
单纯的采集能力不构成壁垒。从自研采集设备、标准化任务体系、云端数据基础设施到标注质检平台的全链路能力，才是真正难以复制的资产。

---

## 我们做什么

<CardGroup cols={3}>
  <Card title="数据采集" icon="video">
    使用自研 CyberCap / CyberGlove 可穿戴设备，在家庭和工业场景下进行标准化 EgoCentric 数据采集
  </Card>
  <Card title="标注质检" icon="tag">
    通过蓝鸟云平台完成 GV 标注和 Short-segment 精细标注，确保每条数据可溯源、符合训练规范
  </Card>
  <Card title="结构化交付" icon="database">
    经由 Grace 数据库和 Cloud Tracking System 管理数据全生命周期，支持多客户的规模化交付
  </Card>
</CardGroup>

---

## 系统架构层次

CyberOrigin 的技术体系按七层架构组织：

| 层级 | 名称 | 职责 |
|---|---|---|
| L1 | 传感器与采集硬件 | CyberCap、CyberGlove 设备 |
| L2 | 边缘预处理 | 本地缓冲、实时压缩、数据校验 |
| L3 | EgoCentric 多模态采集 | 视频流、IMU、手部追踪同步 |
| L4 | 云端数据管道 | 上传、转码、存储、去重 |
| L5 | 云端 QC 与标注 | 蓝鸟云质检、短段标注、入库 |
| L6 | 数据基础设施 | Grace DB、Cloud Tracking、查询服务 |
| L7 | 模型训练消费 | Policy Training、世界模型训练 |

详细架构见 [系统架构总览](/docs/guides/system-architecture)。

---

## 当前阶段

**2026 Q2 — 基础设施收尾 + 商业化起跑**

正在完成数据采集→标注→交付完整链路的工程化，同时验证 Policy Training 可行性，建立对外交付节奏。

产品路线图见 [Roadmap](/docs/company/roadmap)。
