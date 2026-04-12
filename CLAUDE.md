
# WebKB — 网页知识库发布项目

将 Obsidian 笔记编译为 Mintlify 可发布的网页知识库。

---

# 目录结构

```
WebKB/
├── drafts/               # 草稿区：自由写作，Obsidian 风格
├── docs/                 # 发布区：Mintlify 兼容格式（这是部署源）
│   ├── guides/           # 教程/指南类
│   ├── references/       # 参考文档类
│   └── blog/             # 博客/随笔类
├── assets/               # 图片等静态资源
├── mint.json             # Mintlify 站点配置
├── TEMPLATE.md           # 发布文章模板
└── CLAUDE.md             # 本文件
```

数据流向：`drafts/ → docs/ → GitHub → Mintlify 自动部署`

---

# 编译规则

## drafts/ → docs/ 的转换流程

收到「发布」指令时：

1. 读取 drafts/ 中的指定文章
2. 转换为 Mintlify 兼容格式：
   - 添加标准 frontmatter（title, description, icon）
   - 将 `[[双向链接]]` 转为 Mintlify 相对路径链接 `[文字](路径)`
   - 将 Obsidian callout `> [!note]` 转为 Mintlify 组件格式
   - 检查图片路径指向 assets/，使用相对路径
3. 写入 docs/ 对应分类目录
4. 更新 mint.json 的 navigation 配置
5. **处理配套素材**：如果源素材目录中有同名或相关的 `.svg` / `.png` / `.jpg` 等图片文件：
   - 复制到 `WebKB/assets/`（保留原文件名或规范化命名）
   - 在 docs 文章正文中用 `<img src="/assets/文件名" alt="..." />` 嵌入
   - SVG 优先放在文章的架构/流程图章节开头，提供视觉入口

## 写作规范

**drafts/ 中可以自由使用：**
- Obsidian 双向链接 `[[]]`
- Obsidian callout 语法
- 任意格式，无需 frontmatter

**docs/ 中必须满足：**
- 标准 frontmatter（参考 TEMPLATE.md）
- 纯 Markdown + Mintlify 组件语法
- 不使用 Obsidian 特有语法
- 图片引用使用相对路径

---

# 站点配置

修改 `mint.json` 来调整：
- 导航结构（navigation）
- 站点名称、logo、配色
- 页脚链接
- AI 搜索开关

每次新增 docs/ 文章后，同步更新 mint.json 中的 pages 路径。

---

# 行为边界

**可以自由做：** 在 drafts/ 新增和编辑文件、在 docs/ 创建发布版本、更新 mint.json 导航、管理 assets/

**需要确认：** 删除 docs/ 中已发布的文章、修改 mint.json 的站点级配置（名称、主题）

**绝对不做：** 删除 drafts/ 中的草稿（发布后草稿保留作为源）、修改本 CLAUDE.md
