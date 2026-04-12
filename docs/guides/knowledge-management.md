---
title: "知识管理协作方式"
description: "CyberOrigin 如何使用 Obsidian + Claude + Mintlify 构建团队知识体系"
icon: "brain"
---

## 整体架构

CyberOrigin 的知识体系由三层工具协作完成：

```
Obsidian Vault（共享）
    ↓  Claude 读取、整理、生成
内部知识库（本站）
    ↓  Git 同步
Mintlify（自动发布）
```

**分工原则：**
- **Obsidian**：所有原始记录的源头（会议、决策、Sprint 日志、研究笔记）
- **Claude**：批量整理、跨文档关联分析、生成结构化文档
- **WebKB（本站）**：对团队成员发布的可查询知识库

---

## Vault 目录结构

```
CyberOrigin/
├── 00-Company/          # 公司身份层（使命、战略、团队）
├── _Inbox/              # 临时暂存区（先写这里，再整理）
├── 01-Product/          # 产品路线图、PRD、竞品分析
├── 02-Engineering/      # 系统架构、技术决策（ADR）、API 文档
│   └── Decisions/       # 技术决策记录，INDEX.md 是入口
├── 03-Operations/       # 会议记录、周报、OKR、Sprint 日志
├── 04-Knowledge/        # 个人学习、行业研究、Onboarding 材料
├── 05-Releases/         # 版本发布记录
└── 06-Templates/        # 所有笔记模板（_Template- 开头）
```

---

## 日常写作规范

### 命名规范

| 类型 | 格式 | 示例 |
|---|---|---|
| 会议记录 | `YYYY-MM-DD-会议名.md` | `2026-04-13-工程评审.md` |
| Sprint 日志 | `YYYY-WNN-Sprint.md` | `2026-W16-Sprint.md` |
| 技术决策 | `ADR-NNN-标题.md` | `ADR-012-Grace-DB选型.md` |
| 周报 | `YYYY-WNN-周报.md` | `2026-W16-周报.md` |

### 写作流程

**快速记录**：想到什么先写进 `_Inbox/`，不必考虑格式，日期命名即可。

**定期整理**：每周把 `_Inbox/` 里的内容归类到对应目录，Claude 可以辅助完成分类和格式化。

**跨文档引用**：用 Obsidian 双向链接 `[[文件名]]` 建立关联，不要用绝对路径。

---

## Claude 在知识管理中的角色

### 高价值任务示例

**生成周报**
```
读取 03-Operations/Sprint-Logs/ 里本周的 Sprint 日志，
以及 Linear 上本周完成和未完成的任务，
生成本周周报，存入 03-Operations/Weekly-Reports/。
```

**整理会议记录**
```
读取今天的会议记录 2026-04-13-XXX.md，
提取所有 Action Items，按负责人整理成表格追加到文件末尾。
```

**生成 Onboarding 文档**
```
读取 02-Engineering/Decisions/INDEX.md 和所有 ADR 文件，
生成给新成员的技术背景指南，用"我们为什么这样做"的语气，
按第一周会遇到的问题组织内容。
```

**风险汇总**
```
读取 02-Engineering/risks.md 和本月所有会议记录，
更新风险登记册，标出新增风险和状态变化。
```

### Claude 的操作边界

Claude 可以自由执行：添加标签/链接、在 `_Inbox/` 创建笔记、在笔记末尾追加内容、在指定目录创建周报。

需要确认才执行：修改 `Roadmap.md`、`Architecture.md`、`Vision.md` 等核心文档正文。

---

## 技术决策（ADR）管理

### 为什么要记录决策

决策记录解决团队最常见的知识断层问题："当时为什么这样选？"。有了 ADR：
- 新成员不需要问前辈，直接看文档
- Claude 可以读取决策历史生成 Onboarding 材料
- 废弃的决策保留原因，不会重复踩坑

### ADR 格式

每个决策文件包含：背景、决策结论、选择原因、放弃的选项、影响与后果。详见 `06-Templates/_Template-ADR.md`。

**INDEX.md 是入口**：`02-Engineering/Decisions/INDEX.md` 是所有决策的索引表，新成员和 Claude 都从这里开始读，不用翻全部文件。

### 关键习惯

- **做决策前**：先写 ADR 草稿，讨论后更新状态
- **实施后**：补充"实际结果和教训"——这比决策本身更有价值
- **废弃时**：不删除，标记为 ❌ 并写明原因

---

## WebKB 发布流程

Obsidian 内容到公开知识库的发布路径：

1. 在 Vault 的对应目录完成内容（Obsidian 格式）
2. 整理为 Mintlify 文档格式（frontmatter + MDX 语法）
3. 存入 `WebKB/docs/` 对应子目录
4. 更新 `WebKB/mint.json` 的 navigation 配置
5. Git commit & push → Mintlify 自动部署

每天早上 7 点有自动化任务扫描 `WebKB/drafts/` 目录，将待发布草稿编译并推送。如需手动触发，运行：

```bash
cd /path/to/Obsidian Vault/WebKB
git add . && git commit -m "docs: update knowledge base" && git push origin main
```

---

## 常见问题

**笔记应该存在 Vault 还是直接写 WebKB？**
原始记录、会议笔记、草稿都应先在 Vault 写。WebKB 发布的是经过整理、适合团队查阅的版本。

**Vault 多人协作怎么同步？**
通过 Git 同步（pull before editing, push after editing）。冲突少见，因为每人主要负责各自目录。

**Claude 能读取 Vault 里的所有文件吗？**
是的，通过 obsidian-mcp 连接后，Claude 可以读取和写入 Vault 的所有文件，遵循 CLAUDE.md 里定义的操作边界。
