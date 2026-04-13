---
title: "蓝鸟云质检平台"
description: "CyberOrigin 视频标注与质检工具：任务分段操作、不合格原因分类与键盘快捷键"
icon: "cloud"
---

## 概述

蓝鸟云是 CyberOrigin 数据采集流水线中的视频标注和质检平台，质检岗和标注岗的日常工作在此执行。平台支持两类核心操作：**任务分段**和**不合格原因标注**。

## 快捷键

| 键 | 功能 |
|---|---|
| `J` | 任务分段（开始 / 结束） |
| `L` | 标注不合格片段（开始 / 结束） |

## 任务分段（J 键）

<Steps>
  <Step title="开始分段">
    第一次按 `J`，选择对应的**场景**和**任务名称**，系统记录当前时刻为起始帧。
    
    **大拇指手势** = 任务开始信号，标记为初始帧。
  </Step>
  <Step title="结束分段">
    第二次按 `J` 并确认，完成当前任务的分段标注。
    
    **二指手势** = 任务结束信号，标记为结束帧。
  </Step>
</Steps>

## 不合格标注（L 键）

质检过程中，发现不符合规范的片段用 `L` 键标注起止时间，并在结束时选择对应的不合格原因。

### 家庭场景（5 类）

| 代码 | 说明 |
|---|---|
| `image_flicker` | 画面频闪（播放时画面持续亮暗闪动） |
| `operator_action_too_fast` | 操作员动作异常快 |
| `operator_hand_static` | 操作员手静止超过 5 秒 |
| `illegal_use_props` | 违规使用道具 |
| `other_unqualified` | 其他不合格 |

### 超市场景（17 类）

在家庭场景 5 类基础上，额外增加：

| 代码 | 说明 |
|---|---|
| `target_out_of_frame` | 操作目标出画面 |
| `face_appeared` | 出现未遮挡人脸 |
| `multiple_hands_appeared` | 画面中超过两只手 |
| `part_of_hands_out_of_frame` | 部分手出画面 |
| `hands_out_of_frame` | 双手出画面 |
| `push_shopping_cart` | 推购物车时手出画面 |
| `task_definition_and_items_not_related` | 任务定义与交互物品不相关 |
| `image_blur` | 画面模糊 |
| `mirror_problem` | 镜子现象 |
| `phone_appeared` | 出现手机 |
| `chinese_appeared` | 画面中出现中文 |
| `operator_unpleasant_behavior` | 操作人员不雅行为 |

## 延伸阅读

<CardGroup cols={2}>
  <Card title="EgoCentric 数据标注" icon="tag" href="/zh/references/egocentric-annotation">
    标注方法论与输出格式
  </Card>
  <Card title="标注团队 KPI" icon="chart-bar" href="/zh/references/annotation-team-kpi">
    质检岗产出基准（19 小时/日）
  </Card>
  <Card title="数据采集操作规范" icon="video" href="/zh/guides/data-collection-ops">
    采集侧规范，与质检标准对应
  </Card>
  <Card title="标注流水线" icon="tags" href="/zh/references/annotation-pipeline">
    两层标注结构与 QC 缺陷代码表
  </Card>
</CardGroup>
