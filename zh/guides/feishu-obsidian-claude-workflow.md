---
title: "飞书+Obsidian+Claude 三件套工作流"
description: "团队管理中枢的实战架构：让人只用飞书，Obsidian 和 Linear 退到后台由 Claude 操作"
icon: "diagram-project"
---

> 飞书的对话到工作决策，其实就在沟通之中、做事之中——这个沟通发生在每日每一个瞬时。把每个瞬时的片段都记录下来。
>
> —— 2026-04-26 即时口述

## 起点

中型团队（30+ 人）的工作流困境：

- **飞书**是真实工作发生的地方（消息、会议、客户群）
- **Linear** 只有少数管理者维护，团队成员极少打开
- **Obsidian** 完全没用起来——"打开 Obsidian 写笔记 → 整理 → 更新 Linear" 这条路径推不动

CyberOrigin 的反转命题：

> 不要让大家使用 Obsidian 和 Linear，让系统自己用。人只需要做他们本来就在做的事——在飞书说话、开会、发消息。

## 三件套定位

| 层 | 工具 | 角色 |
|---|---|---|
| 沟通入口 | **飞书** | 消息 / 会议 / 客户群 |
| 执行单元 | **Linear** | Issue / Project / Cycle |
| 决策追溯 + 跟踪管控 | **Obsidian Vault**（含 GH Actions 自动化） | OKR / 跨层决策 / 风险 / 节奏型记录 |

各自不互相替代。Claude 在中间扛重活：读飞书消息 → 分类（4 象限）→ 写到 vault + Linear。

```
飞书消息（沟通入口）
   │
   ├─ 决策类     ──→ Obsidian DEC 文档（24h 编辑窗口）
   ├─ 阻塞行动项  ──→ Linear Issue（带层级标签 + SLA）
   ├─ 进展类     ──→ Linear Issue 状态更新
   └─ 噪音      ──→ 忽略

GH Actions cron 自动产出
   ├─ 每日 09:00 上海 → Linear 前看看板
   ├─ 每周一 07:00    → Linear 风险周报（5 维度评分）
   └─ 每周五 18:00    → 跨层周报（决策/阻塞/修改文件）

Claude session 入口
   └─ 进 session 时主动检查任务清单 → 跑未完成的 daily triage
```

## Vault 4 工作目录

经过两轮精简，最终落定：

```
team-vault/
├── 00-dashboard/      管理层每日入口（永久 + cron 看板）
├── 10-decisions/      跨层决策 DEC-XXX（追溯权威）
├── 30-cycles/         节奏型回看（按数据源 + 周期分子目录）
├── 40-integrations/   集成配置层（飞书 / Linear 子目录）
└── 90-archive/        软删除区
```

每个目录单一职责。"按数据源分子目录"是关键——`30-cycles/feishu/daily/` 和 `30-cycles/linear/daily/` 平级，不必靠文件名后缀（`-feishu-triage`）区分。扩展新数据源（如 GitHub）只需新建 `30-cycles/{source}/...`，不破坏现有结构。

## 永久入口 vs cron 看板分离

`00-dashboard/` 含两个文件，**单职责分离**：

| 文件 | 角色 | 维护 |
|---|---|---|
| `daily-watch.md` | 永久入口（任务自检 + 数据源指针 + 今日高优 + 决策候选） | 人 + Claude session 维护，**不被 cron 覆盖** |
| `linear-watch.md` | Linear 前看看板（今日紧急/逾期/无标签） | GH Actions cron 每日 09:00 重写 |

cron 写的内容（前看看板）独立一个文件，避免和人维护的内容（任务/决策）混在一起。

## 每日工作流

```
1. 团队负责人进 Claude session
2. Claude 读 daily-watch.md 任务自检 → "feishu daily 未跑"
3. 人："OK 跑" → Claude 执行 daily triage runner
   ├─ Runner 拉飞书 7 个监控群昨日消息（约 100 条）
   ├─ Pre-filter：跳过单字回复/打卡/媒体（保留 ~40%）
   ├─ Sonnet 4.6 batch 分类（A 决策 / B 进展 / C 阻塞 / D 噪音 + 高优标记）
   ├─ 写完整日报到 vault
   └─ 更新 daily-watch.md 4 个 marker（数据源时间戳 / 高优 Top 3 / 决策候选）
4. 人看 daily-watch：
   ├─ 🔥 今日高优 Top 3 → 决定立即 take action
   └─ 📝 今日决策候选 → 选「升级正式 DEC」/「仅记录」/「忽略」
```

整个流程一杯咖啡时间内完成（5-10 分钟）。

## 每周工作流

### 周一上午

- 跑飞书周报（基于过去 7 天 daily 自动汇总）
- Claude 拉 Linear 本周 Sprint 进度按项目分组
- 看 Linear 风险周报（cron 已写）→ 决定本周重点

### 周五傍晚

- GH Actions 自动产出跨层周报（决策 + 跨层阻塞 + 本周修改文件）
- 周一早上回看，确认本周收尾

## 关键决策（实战经验）

### 关键词分类不够，必须用 LLM

最初设计用关键词字典分类（"决定/拍板"→A 决策、"完成了"→B 进展、"卡住"→C 阻塞）。一周实测：

- **Keyword**：A 决策 = 0，高优 = 0，D 噪音 = 83%
- **Sonnet 4.6 LLM**：A 决策 = 6，高优 = 11，D 噪音 ~ 6
- **分歧率：82%**

中文 IM 的语境特点：决策不说"决定"，直接说结论；紧急不说"urgent"，直接说时间。**关键词是语法分类器，LLM 是语义分类器**。中文 IM 信号几乎全在语义层。

成本对比：每天 350 条消息，单次 LLM 分类 ~$0.05，月成本 ~$5（开 prompt cache 后可降至 ~$1.5）。

### 决策候选模式：runner 注入 + 人拍板

LLM 抓到的"决策语" 不一定是真的决策（可能只是技术拍板，记录即可）。所以 runner 跑完后，把当日 A 类前 3 条注入 daily-watch，**每条给人 3 选项**：

```
**候选 N**: [群名] [time] 发件人: 原文摘要
   → 选项: ✅ 升级正式决策（让 Claude 起草）/ ⏸️ 仅记录 / ❌ 忽略
```

人选「升级」后，Claude 才起草决策文档。**runner 不自动起草** —— 决策是"长期承诺"，必须人亲自拍板才能升级。这跟 MVP 阶段的"草稿+👍 确认" 信任红线一致。

### 简单可用 > 完美复杂

实战中三次降级：

| 场景 | 复杂方案 | 降级到 | 触发因素 |
|---|---|---|---|
| 飞书接入 | webhook + Worker + 鉴权中转 + 路由 | 用官方 CLI 直连 | CLI 让整个 Infra 层不必要 |
| 决策落地 | 写 DEC + 建 Linear Project + 拆 8 Issue | DEC + 取消旧 + 直接动手 | 决策是"反转/精简"而非"新方向"，铺工程容易拖死 |
| 自动触发 | macOS launchd 每日 09:15 cron | session 触发（人主动一句"OK 跑"） | launchd 进程调 API 持续 403，3 次诊断未定位根因 |

**当工程方案遇到难以快速定位的阻力时，优先降级到简单模式**而不是继续 debug，除非 ROI 明显值得。这条原则一次 session 内救了 3 次。

## 设计原则总结

1. **简单可用 > 完美复杂** — 难解阻力时主动降级
2. **永久 vs 自动分文件** — 不同维护方的内容放不同文件
3. **按数据源分子目录** — 不靠文件名后缀区分数据源
4. **配置 source of truth 在 vault** — workflow yml 里的 prompt 是副本
5. **关键判断要人拍板** — LLM 提交候选，不自动升级长期承诺
6. **历史快照不改** — 结构重组时保留当时事实
7. **路径迁移三档分类** — live link 跟着改 / historical snapshot 不改 / 别人项目 CLAUDE 别擅自动
8. **架构反转走精简流程** — 既有方案精简不必新建 Linear Project

## 现场实测：从硬件讨论到 Linear Issue

2026-04-27 中午一段 8 分钟的现场记录，演示了"会议进行 → AI 实时同步 → 自动到 Linear Issue"的完整链路。试用过程暴露了 3 条真实的产品体验：

### 1. "慢"不等于"卡"

> "唯一的问题就是它会一直那啥——就是太慢了。"
> "啥叫慢呢？你不能总指望咔就出来了，它更多的问题是推理分析问题。"

LLM 的"慢"是推理决策时间，不是网络或系统卡顿。把"在思考、在决策、在分析"显式回显给用户，胜过一个空 spinner——后者会让人误以为系统挂掉。

### 2. MCP 取代 function calling

> "leader 提供了 MCP 模型上下文接口。"
> "之前还是 function calling。"

三件套的 Claude → Linear 链路在 2026 Q2 全切到 MCP（OAuth 接入），告别了每次手动 token 维护。这是工作流真正能跑起来的底层前提之一。

### 3. 数据流转的真实闭环

> "他应该推送给这个人，这个人再去回看一遍，然后再回传。把 AI 写的项目方案，让大家自己去学习。"

人不是被 AI 替换，而是被 AI **推送到关键决策点**。AI 写初稿 → 人回看修订 → 反馈回传 → 形成可学习的项目方案库。这正是文章开头那句"日常对话即工作决策记录"的实际落地路径。

> 来源：现场口述，2026-04-27 12:52

## 适用边界

本文工作流的前提：

- 团队有 30+ 人但管理者 ≤ 5 人（决策可以由少数人拍板）
- 飞书是公司主要沟通工具
- 已经在用 Linear（或类似 Issue tracker）
- 知识库用 Obsidian（或类似 markdown vault）
- 至少一个管理者懂用 Claude（CLI 或 Code 都行）
- 月预算 ~$5-30 用于 LLM 分类

不适用：完全不用 Linear 的团队、不用飞书的国际团队、超大型组织（千人级）。

## 相关

- 三件套设计原则：[Knowledge Management](/zh/guides/knowledge-management)
- 系统架构总览：[System Architecture](/zh/guides/system-architecture)
- 渐进式落地：[Phased Rollout](/zh/guides/phased-rollout)
- 低摩擦记录：[Low-Friction Inbox](/zh/guides/low-friction-inbox)
