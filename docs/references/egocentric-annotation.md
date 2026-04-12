---
title: "EgoCentric 数据标注"
description: "第一人称视频标注方法论：GV 标注、Short-segment 标注、任务类型覆盖与输出格式规范"
icon: "tag"
---

## 定义

针对第一人称（EgoCentric）视角采集的视频数据，按照预定义的步骤流程进行分段和动作标注的方法论。目标是从完整视频中准确识别每段独立任务，并在每段中按步骤顺序标注交互动作。

<Warning>
标注必须严格遵循既定步骤顺序。步骤缺失、顺序错误或与标准流程不一致的片段视为**无效**，不计入产出。
</Warning>

## 标注层次

标注分为两个层次，依序执行：

**GV 标注** → **Short-segment 标注**

### GV 标注

<Steps>
  <Step title="校准段标注">
    每段视频开头为校准动作（缓慢走动、静止不动、手部不自然出镜）。标注员找到校准结束帧并标注，允许结束帧略微靠后，但之后必须只有执行任务的动作。
  </Step>
  <Step title="动作切分点标注">
    标注自然的动作切分点，即上一个动作完成与下一个动作开始之间的任意帧。相邻切分点间隔不超过 1 分钟，无需语言标注。
  </Step>
</Steps>

### Short-segment 标注

每次与物品发生交互动作都需要标注，时长通常 **2-5 秒**，标注之间允许存在 gap。

每条标注包含两个字段：

| 字段 | 说明 | 示例 |
|---|---|---|
| **Skill** | 描述核心动作的动词 | `pick`, `place`, `hold`, `retrieve` |
| **Description** | 谓语 + 宾语 + 方式 + 空间关系 | `Place the held pear into the plastic bag in the shopping bag.` |

<Tip>
动词和描述句式可参考 [HD-EPIC Dataset](https://hd-epic.github.io/)。
</Tip>

## 任务类型覆盖

当前定义了 **16 类家庭场景任务**：

| # | 任务 | # | 任务 |
|---|---|---|---|
| 1 | 洗菜切菜 | 9 | 整理冰箱 |
| 2 | 厨房商品分类 | 10 | 整理玩具 |
| 3 | 收拾餐桌 | 11 | 整理工具 |
| 4 | 清洗餐具 | 12 | 整理工具箱 |
| 5 | 沏茶 / 泡咖啡 | 13 | 制作沙拉 |
| 6 | 整理枕头 / 床被 | 14 | 制作零食袋 |
| 7 | 整理衣服 / 袜子 | 15 | 折叠衣物 |
| 8 | 整理书架 / 桌面 | 16 | 扫地拖地 |

自建工厂后新增工业场景：电脑主机拆装、帐篷拆搭、家具拆装、自行车拆装。

## 输出格式

标注结果输出为 `.json`，包含三个部分：

```json
{
  "video_info": {
    "name": "GX010755.MP4",
    "total_frames": 9419,
    "fps": 29.97,
    "annotator_id": "cyber001"
  },
  "annotations": [
    {
      "start_frame": 95,
      "end_frame": 256,
      "valid": true,
      "content": "place bags on the table"
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

- `video_info`：视频元信息，系统自动填写
- `annotations`：每个有效片段内的动作标注，顺序必须与步骤流程一致
- `segmentations`：每条完整数据片段的起始帧和结束帧

## 相关文档

- [数据采集操作规范](/docs/guides/data-collection-ops) — 校准流程和 Session 结构
- [蓝鸟云质检工具](/docs/references/laniao-cloud) — 执行质检和标注的平台
- [标注团队 KPI](/docs/references/annotation-team-kpi) — 产出基准和质量合格线
