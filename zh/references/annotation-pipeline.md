---
title: "Annotation Pipeline"
description: "How egocentric video is annotated: two-layer segmentation, action labeling, QC defect codebook, and JSON output format"
icon: "tags"
---

## 概览

一段采集视频中可能包含多个合规的采集任务。标注流水线先将完整视频切分成独立任务，再对每个片段按既定步骤流程进行逐步动作标注。

步骤缺失、顺序错误或与流程不一致的片段会被标记为无效，不参与下游训练。

## 两层标注

### 第 1 层：GV 标注（全视频）

**标定段标注** — 每段视频开头都是标定序列（慢速走动、双手不入画、静止姿态）。标注员标出标定结束帧，该帧之后全部是任务数据。

**动作切分** — 标注员标记自然的动作边界（一个动作完成与下一个动作开始之间的过渡）。相邻边界之间不超过 60 秒。本层不需要语言标签——仅做时间切分。

### 第 2 层：Short-segment 标注（精细层）

每次与物品的交互对应一个独立的标注片段，时长通常 2-5 秒。片段之间允许存在 gap。

每条片段包含两个字段：

- **Skill**：描述核心动作的单个动词（`hold`、`retrieve`、`pick`、`place`、`push`、`fold` 等）
- **Description**：遵循 `动词 + 宾语 + 方式 + 空间关系` 模式的结构化句子

示例：*"Place the held pear into the plastic bag in the shopping bag."*

空间关系既要描述物体之间的关系（从 A 位置到 B 位置），也要描述精确的位置上下文（例如"放进购物车里方便面旁边的位置"）。

动词和宾语词表参考 [HD-EPIC dataset](https://hd-epic.github.io/)。

## 任务库

当前定义了 16 类家庭任务，每类都有明确的步骤流程和起止帧判定标准：

| ID | 任务 | 关键约束 |
|---|---|---|
| 1 | 厨房物品分类 | 完整流程：从袋子到货架到冰箱 |
| 2 | 沏茶 | 包含清理：去卫生间洗茶具，放回桌面 |
| 3 | 冲咖啡 | 清理方式与沏茶相同 |
| 4 | 整理衣物 | 至少 5 件；起始为杂乱一堆，结束为叠好一摞 |
| 5 | 整理袜子 | 收集、折叠、装盒、收纳、关柜 |
| 6 | 整理床铺 | 换枕套和床单，整齐折叠 |
| 7 | 厨房清洁 | 灶台、水槽、台面、橱柜、抹布清理 |
| 8 | 冰箱清洁 | 清空内容物与搁架，清洗，内壁擦拭，复位 |
| 9 | 客厅收拾 | 地面物品、沙发靠垫、茶几 |
| 10 | 清洗餐具 | 收集、入槽、清洗、沥干、分类归位 |
| 11 | 扫地拖地 | 带工具入场、扫、收灰、拖、归位 |
| 12 | 工具箱整理 | 收集、补齐缺件、装盒、收纳 |
| 13 | 制作沙拉 | 转移蔬菜、搅拌、调味、清理 |
| 14 | 零食分装 | 开袋、舀取、倾倒、封口、收纳 |
| 15 | 折叠衣物 | 平铺、左袖/边、右袖/边、从下往上翻折 |
| 16 | 桌面整理 | 笔、笔筒、杯子、盒子——各归指定位置 |

工业任务（工厂场景）正在新增：电脑主机拆装、帐篷搭设、家具组装、自行车组装。

## 质量控制

QC 在名为"蓝鸟云"的平台上执行，两类操作均为键盘驱动：

**任务切分（J 键）**：第一次按下选择场景 + 任务名，并标记起始帧。第二次按下确认结束帧。采集员视觉提示：大拇指手势 = 任务开始，二指手势 = 任务结束。

**不合格标注（L 键）**：第一次按下标记不合格起始。第二次按下从代码表中选择不合格原因。

### 不合格代码表

家庭场景（5 类）：

| 代码 | 触发条件 |
|---|---|
| `image_flicker` | 播放时画面持续亮暗波动 |
| `operator_action_too_fast` | 动作异常快 |
| `operator_hand_static` | 手部静止超过 5 秒 |
| `illegal_use_props` | 违规使用道具 |
| `other_unqualified` | 其他不合格 |

超市场景在此基础上增加 12 类：`target_out_of_frame`、`face_appeared`、`multiple_hands_appeared`、`part_of_hands_out_of_frame`、`hands_out_of_frame`、`push_shopping_cart`（推车时手出画面）、`task_definition_and_items_not_related`、`image_blur`、`mirror_problem`、`phone_appeared`、`operator_unpleasant_behavior`、`chinese_appeared`。

## 输出格式

标注结果导出为 JSON，包含三个顶层字段：

```json
{
  "video_info": {
    "name": "GX010755.MP4",
    "total_frames": 9419,
    "fps": 29.97,
    "annotator_id": "cyber001",
    "duration": 314.28,
    "codec": "hevc",
    "shape": [1080, 1920, 3]
  },
  "annotations": [
    {
      "start_frame": 95,
      "end_frame": 256,
      "start_timestamp": 4170.833,
      "end_timestamp": 8575.233,
      "valid": true,
      "content": "place bags on the table",
      "recheck": false
    }
  ],
  "segmentations": [
    {
      "start_frame": 8,
      "end_frame": 5499,
      "valid": true,
      "env": "kitchen",
      "task": "unpack_groceries"
    }
  ]
}
```

- `video_info` 由文件元数据自动生成
- `annotations` 按步骤顺序排列——顺序错误的条目会导致整段失效
- `segmentations` 定义任务边界，并给出环境与任务类型标签

## 延伸阅读

<CardGroup cols={2}>
  <Card title="EgoCentric 数据标注" icon="tag" href="/zh/references/egocentric-annotation">
    标注方法论的中文详解
  </Card>
  <Card title="蓝鸟云质检平台" icon="cloud" href="/zh/references/laniao-cloud">
    标注和质检工具的快捷键与工作流
  </Card>
  <Card title="标注团队 KPI" icon="chart-bar" href="/zh/references/annotation-team-kpi">
    产出基准与奖金结构
  </Card>
  <Card title="数据采集操作规范" icon="video" href="/zh/guides/data-collection-ops">
    上游采集规范
  </Card>
</CardGroup>
