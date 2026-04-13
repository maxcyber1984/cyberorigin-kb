---
title: "新成员入职指南"
description: "加入 CyberOrigin 的第一周：系统了解、工具接入、协作规范"
icon: "user-plus"
---

## 欢迎加入

CyberOrigin 是一家专注于**具身智能数据基础设施**的公司，为机器人和具身智能系统提供高质量的 EgoCentric（第一人称视角）训练数据。本指南帮助你在第一周快速了解团队的工作方式。

---

## 第一天：了解公司

先读这三份文档，建立基础认知：

1. [公司概览](/zh/company/overview) — 我们是谁、做什么、核心技术判断
2. [产品路线图](/zh/company/roadmap) — 当前阶段目标和本季度里程碑
3. [系统架构总览](/zh/guides/system-architecture) — 七层技术架构和数据流

**关键概念速览：**

- **EgoCentric 数据**：第一人称视角的人类动作数据，是训练机器人策略模型的核心原料
- **GV（Glove Video）**：手套+视频同步采集的数据类型，用于精细手部动作建模
- **Short-segment 标注**：对 GV 视频进行精细时序切割和分类的标注任务
- **Grace DB**：公司自研的核心数据库，管理所有采集和标注数据
- **Cloud Tracking System**：视频数据的全链路追踪系统

---

## 第二天：工具接入

### 必备工具

| 工具 | 用途 | 获取方式 |
|---|---|---|
| Linear | 任务管理、Sprint 追踪 | 联系 Max 邀请 |
| Obsidian | 知识库、会议记录、文档 | 下载 + 接入共享 Vault |
| 蓝鸟云 | QC 标注平台 | 联系 Shiqi 开通账号 |
| GitHub | 代码和文档版本管理 | 联系工程负责人 |

### Obsidian Vault 接入

团队知识库通过 Obsidian Vault + Git 共享。接入步骤：

```bash
# 克隆 Vault（联系 Max 获取仓库地址）
git clone <vault-repo-url>

# 使用 Obsidian 打开该文件夹
# File → Open Vault → 选择克隆的文件夹
```

Vault 目录结构说明见 [知识管理协作方式](/zh/guides/knowledge-management)。

---

## 第三天：数据采集流程

CyberOrigin 的核心业务是 **EgoCentric 数据采集 → 标注 → 交付**，请优先了解以下流程：

**采集阶段**
- 采集人员佩戴 CyberCap（头戴摄像头）和 CyberGlove（手套传感器）
- 在标准化场景（家庭 16 类任务 / 工业场景）下执行操作任务
- 数据通过边缘预处理后上传至云端

**标注阶段**
- 在蓝鸟云平台完成 GV 标注（视频分段）和 Short-segment 精细标注
- 标注规范见 [EgoCentric 标注规范](/zh/references/egocentric-annotation)
- 蓝鸟云操作指南见 [蓝鸟云标注平台](/zh/references/laniao-cloud)

**交付阶段**
- 标注完成后数据自动写入 Grace DB
- 通过 Cloud Tracking System 管理交付状态
- 按客户需求导出（HDF5 / JSON 等格式）

---

## 第四天：工程规范

### 技术决策查询

重大技术决策记录在 `02-Engineering/Decisions/` 目录下（ADR 格式）。遇到"为什么这样做"的问题，先查 INDEX.md。

### 常见问题

**为什么用 TOS 而不是 Aliyun OSS？**
Q2 2026 已完成从 Aliyun OSS 到 TOS 的视频迁移，成本和性能综合评估后的决策。

**Grace DB 是什么数据库？**
Grace 是自研的数据管理系统，基于关系型数据库构建，支持视频元数据查询和标注状态追踪。具体接口见 `02-Engineering/API/` 文档。

**数据标注 QA 标准是什么？**
参考 [EgoCentric 标注规范](/zh/references/egocentric-annotation) 中的质检标准和 17 项不合格原因代码。

---

## 第一周 Checklist

```
□ 读完公司概览、路线图、系统架构
□ Linear 账号激活，加入对应 team
□ Obsidian Vault 本地克隆成功
□ 蓝鸟云账号开通（如涉及标注工作）
□ 了解当前 Sprint 目标（Linear → Cycles）
□ 与直接负责人完成第一次 1on1
□ 在 Vault 的 _Inbox/ 写下你的第一条笔记
```

---

## 需要帮助？

- **产品/战略方向**：找 Max
- **数据标注/交付**：找 Shiqi
- **工程问题**：找 chengpei
- **硬件/设备**：找 lizi
- **MoCap/动捕**：找 Yu Liang

## 延伸阅读

<CardGroup cols={2}>
  <Card title="知识管理协作方式" icon="brain" href="/zh/guides/knowledge-management">
    Vault 目录结构与写作规范
  </Card>
  <Card title="公司概览" icon="building" href="/zh/company/overview">
    第一天要读的公司背景
  </Card>
  <Card title="系统架构总览" icon="layer-group" href="/zh/guides/system-architecture">
    七层架构和数据流
  </Card>
  <Card title="产品路线图" icon="map" href="/zh/company/roadmap">
    当前阶段目标和本季度里程碑
  </Card>
</CardGroup>
