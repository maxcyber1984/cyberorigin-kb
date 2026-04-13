---
title: "Session-to-Mintlify Pipeline"
description: "Agent workflow for automatically compiling Claude conversations into Mintlify docs and deploying via GitHub"
icon: "arrows-rotate"
---

## 为什么要做这个

手工把 Claude 对话整理成发布文档是瓶颈。对话本身已经是结构化思考——"我把这件事讲给了 Claude"到"它出现在文档站上"之间，不应该还要耗费几个小时的手工编辑。

这条管道把这段差距压缩成三个阶段：**抽取 → 转换 → 同步**。

## 架构

<img src="/assets/session_to_mintlify_agent_flow.svg" alt="Session to Mintlify agent flow diagram" />

### 阶段 1：Session 捕获

Claude 目前还没有官方的对话导出 API，有三种可行输入：

- **手动粘贴**：把对话复制到 `.txt` 或 `.json`，适合偶尔的一次性导入
- **API 原生**：如果你直接用 Claude API，`messages[]` 数组本身就是结构化的——直接喂给抽取器即可
- **浏览器插件**：类似"Save ChatGPT"这类插件可以抓取 claude.ai 并输出 JSON

### 阶段 2：抽取 Agent

一次带系统提示的 Claude API 调用，负责把对话解析成结构化文档。核心提示的思路：

```
You are a technical documentation extractor. Given a conversation, identify:
- Each distinct topic (one Mintlify page each)
- Title, description, body sections
- Code blocks, tables, step lists
- Category (edge/cloud/processing/storage/blog/)

Output strict JSON:
{
  "pages": [
    {
      "path": "cloud/kafka-broker",
      "title": "...",
      "description": "...",
      "sections": [...]
    }
  ]
}
```

拿到 JSON 输出后，再经过一个 MDX Writer 函数，把 sections 转换成 `.mdx` 文件，并使用合适的 Mintlify 组件（`<Steps>`、`<Card>`、`<Warning>` 等）。

### 阶段 3：GitHub 同步

GitHub Contents API 允许通过一次 PUT 请求创建或更新文件：

```python
import base64, requests

def push_to_github(path, content, token, repo):
    url = f"https://api.github.com/repos/{repo}/contents/docs/{path}"
    existing = requests.get(
        url,
        headers={"Authorization": f"token {token}"}
    ).json()
    sha = existing.get("sha")  # required when updating existing files

    requests.put(url, json={
        "message": f"docs: sync {path} from Claude session",
        "content": base64.b64encode(content.encode()).decode(),
        "sha": sha  # omit for new files
    }, headers={"Authorization": f"token {token}"})
```

PUT 会触发 GitHub App 的 webhook 通知 Mintlify，后者执行构建并部署。从"对话结束"到"页面上线"之间不再需要人工介入。

## Orchestrator 选型

三种方向，复杂度权衡各不相同：

| 方案 | 适用场景 | 复杂度 |
|---|---|---|
| **Python 脚本** | 首次端到端验证、手动触发 | 低 |
| **n8n**（自托管） | 可视化流程、无代码触发器（cron、webhook） | 中 |
| **LangGraph** | 复杂 Agent 链、重试逻辑、条件分支（"这是 blog 还是 docs？"） | 高 |

**推荐路径**：先用 Python 脚本端到端跑通，再根据是否需要定时或 webhook 触发决定是否上 n8n。除非前两者撞墙，否则不要过早跳到 LangGraph——过早引入 Agent 复杂度是一个常见陷阱。

## 待定设计问题

- **抽取粒度**：一段涉及多个话题的对话如何切分？每个子话题一页 vs. 每段对话一页？
- **人工复核**：草稿 PR 模式（提交到分支、开 PR）vs. 直接推送 main？前者更安全但增加摩擦
- **版本管理**：相同话题在多段对话中反复出现时，合并还是覆盖？
- **组件选择**：用什么启发规则决定 `<Steps>` vs. 普通有序列表，或 `<Warning>` vs. 普通引用？

## 关键风险

- 抽取器的分类准确率直接决定导航可用性——一个归错类的页面比没有页面更糟
- 对话的信噪比低于精修过的草稿，面向公开的内容不能跳过人工复核
- GitHub API 限流需要关注批量同步场景（认证用户 5000 req/h，但真正的瓶颈其实是 Mintlify 的构建队列）

## 延伸阅读

<CardGroup cols={2}>
  <Card title="知识管理协作方式" icon="brain" href="/zh/guides/knowledge-management">
    Obsidian + Claude + Mintlify 的整体协作
  </Card>
  <Card title="新成员入职指南" icon="user-plus" href="/zh/guides/onboarding">
    团队知识体系的接入流程
  </Card>
</CardGroup>
