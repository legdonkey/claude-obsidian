---
source: docs/install-guide.md
source_version: 1.9.2
translated_at: 2026-06-10
---

# claude-obsidian：安装指南

**Claude + Obsidian 知识助手**
版本 1.9.2 · 公开权威仓库：[github.com/AgriciDaniel/claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian) · 社区抢先体验镜像（Pro）：[AI Marketing Hub org](https://github.com/AI-Marketing-Hub)

> ℹ️ 下面的安装命令使用的是**公开开源**地址（`AgriciDaniel/claude-obsidian`），推荐所有人使用，无需任何会员资格。如果你是 [AI Marketing Hub Pro](https://www.skool.com/ai-marketing-hub-pro) 会员、希望抢先体验开发中的功能，可以把所有 `AgriciDaniel/claude-obsidian` 替换为 `AI-Marketing-Hub/claude-obsidian`，并把插件标识 `claude-obsidian@agricidaniel-claude-obsidian` 替换为 `claude-obsidian@ai-marketing-hub-claude-obsidian`。

> **可选：DragonScale Memory 扩展。** 如果你想要扁平的抽取式日志折叠（log folds）、确定性的页面地址、语义切片 lint，以及边界优先的 autoresearch 选题，请在完成基础安装后运行 `bash bin/setup-dragonscale.sh`。除基础安装外还需要的额外前置条件：`flock`（Linux 上标准自带；macOS 上可通过 `util-linux` 获取）和 `python3`（用于切片和边界辅助工具）。可选：如果你想使用语义切片 lint，需要安装 `ollama` 并拉取 `nomic-embed-text`（仅限机制 3；当 ollama 或该模型不可用时会优雅地空跑）。边界优先的评分器（机制 4）只需要 `python3`，不需要 ollama。面向用户的指南见 [`docs/dragonscale-guide.md`](../../docs/dragonscale-guide.md)，完整规格见 `wiki/concepts/DragonScale Memory.md`，1.6.0 版本交付内容见 `CHANGELOG.md`。

---

## 什么是 claude-obsidian？

claude-obsidian 是一个 Claude Code 插件 + Obsidian 仓库，用于构建并维护一个持久、复利式增长的知识库。你添加的每一份来源都会被处理成相互交叉引用的 wiki 页面。你提出的每一个问题都会从已读过的全部内容中提取信息。知识会像利息一样不断复利增长。

基于 Andrej Karpathy 的 LLM Wiki 模式构建。

---

## 前置条件

| 工具 | 如何获取 | 说明 |
|------|--------------|-------|
| **Claude Code** | `npm install -g @anthropic-ai/claude-code` | 提供免费档 |
| **Obsidian** | [obsidian.md](https://obsidian.md) | 免费 |
| **Git** | 大多数系统已预装 | 用于方案 1 |

---

## 安装

### 方案 1：克隆为仓库（推荐）

2 分钟内完成完整设置。

```bash
git clone https://github.com/AgriciDaniel/claude-obsidian
cd claude-obsidian
bash bin/setup-vault.sh
```

然后在 Obsidian 中：**Manage Vaults → Open folder as vault → 选择 `claude-obsidian/`**

在同一个文件夹中打开 Claude Code，输入 `/wiki`。

### 方案 2：作为 Claude Code 插件安装

在 Claude Code 中安装插件分为两步。先添加 marketplace 目录，然后从中安装插件。

```bash
# 第 1 步：添加 marketplace
claude plugin marketplace add AgriciDaniel/claude-obsidian

# 第 2 步：安装插件
claude plugin install claude-obsidian@agricidaniel-claude-obsidian
```

验证安装：
```bash
claude plugin list
```

在任意 Claude Code 会话中：输入 `/wiki`，Claude 会带你完成仓库设置。

### 方案 3：添加到已有仓库

把本仓库中的 `WIKI.md` 复制到你仓库的根目录。然后把以下内容粘贴给 Claude：

```
Read WIKI.md in this project. Then:
1. Check if Obsidian is installed. If not, install it.
2. Check if the Local REST API plugin is running on port 27124.
3. Configure the MCP server.
4. Ask me ONE question: "What is this vault for?"
Then scaffold the full wiki structure.
```

---

## 第一步

### 1. 搭建仓库脚手架

在 Claude Code 中输入 `/wiki`。Claude 会：
- 检测你的仓库模式（website、GitHub、business、personal、research 或 book/course）
- 创建文件夹结构和核心 wiki 页面
- 设置 `wiki/index.md`、`wiki/hot.md`、`wiki/log.md` 和 `wiki/overview.md`

### 2. 放入你的第一份来源

把任意文档放进 `.raw/`：
- PDF、markdown 文件、转录稿、文章、URL

告诉 Claude：`ingest [filename]`

Claude 读取该来源，并创建 8–15 个相互交叉引用的 wiki 页面。

### 3. 提问

```
what do you know about [topic]?
```

Claude 会读取热缓存（hot cache）、扫描索引、深入相关页面，然后给出综合后的答案，并引用具体的 wiki 页面，而不是训练数据。

---

## 命令参考

| 命令 | Claude 做什么 |
|---------|-----------------|
| `/wiki` | 设置检查、搭建脚手架，或从上次中断处继续 |
| `ingest [file]` | 读取来源，创建 8–15 个 wiki 页面，更新索引和日志 |
| `ingest all of these` | 批量处理多份来源，然后交叉引用 |
| `what do you know about X?` | 读取索引 → 相关页面 → 综合出答案 |
| `/save` | 把当前对话归档为一篇 wiki 笔记 |
| `/save [name]` | 以指定标题保存 |
| `/autoresearch [topic]` | 自主研究循环：搜索、抓取、综合、归档 |
| `/canvas` | 打开或创建一个可视化画布 |
| `/canvas add image [path]` | 向画布添加一张图片 |
| `/canvas add text [content]` | 添加一张 markdown 文本卡片 |
| `/canvas add pdf [path]` | 添加一份 PDF 文档 |
| `/canvas add note [page]` | 把一个 wiki 页面钉为带链接的卡片 |
| `lint the wiki` | 健康检查：孤立页面、失效链接、缺口 |
| `update hot cache` | 用最新的上下文摘要刷新 `hot.md` |

---

## 插件（预装）

在 **Settings → Community Plugins** 中启用：

| 插件 | 用途 |
|--------|---------|
| **Calendar** | 右侧栏日历，带字数统计和任务点 |
| **Thino** | 快速备忘捕获面板 |
| **Excalidraw** | 手绘、图片标注 |
| **Banners** | 通过 `banner:` frontmatter 添加头图 |

同样需要从 Community Plugins 安装：

| 插件 | 用途 |
|--------|---------|
| **Dataview** | 驱动仪表盘查询 |
| **Templater** | 根据模板自动填充 frontmatter |
| **Obsidian Git** | 每 15 分钟自动提交仓库 |

---

## CSS 代码片段

`setup-vault.sh` 会自动启用三个代码片段：

| 片段 | 效果 |
|---------|--------|
| `vault-colors` | 在文件浏览器中为 wiki 文件夹标注颜色 |
| `ITS-Dataview-Cards` | 把 Dataview 查询变成可视化的卡片网格 |
| `ITS-Image-Adjustments` | 细粒度的图片尺寸控制；在嵌入后追加 `\|100` |

---

## 六种 Wiki 模式

| 模式 | 适用场景 |
|------|---------|
| **A: Website** | 站点地图、内容审计、SEO wiki |
| **B: GitHub** | 代码库地图、架构 wiki |
| **C: Business** | 项目 wiki、竞争情报 |
| **D: Personal** | 第二大脑、目标、日记综合 |
| **E: Research** | 论文、概念、论文写作 |
| **F: Book/Course** | 章节追踪、课程笔记 |

各模式可以组合使用。

---

## MCP 设置（可选）

MCP 让 Claude 无需复制粘贴即可直接读写仓库笔记。

**方案 A：REST API**

1. 在 Obsidian 中安装 **Local REST API** 插件
2. 复制你的 API key
3. 运行：

```bash
claude mcp add-json obsidian-vault '{
  "type": "stdio",
  "command": "uvx",
  "args": ["mcp-obsidian"],
  "env": {
    "OBSIDIAN_API_KEY": "your-key",
    "OBSIDIAN_HOST": "127.0.0.1",
    "OBSIDIAN_PORT": "27124",
    "NODE_TLS_REJECT_UNAUTHORIZED": "0"
  }
}' --scope user
```

**方案 B：文件系统（无需插件）**

```bash
claude mcp add-json obsidian-vault '{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@bitbonsai/mcpvault@latest", "/path/to/your/vault"]
}' --scope user
```

---

## 故障排查

| 问题 | 修复方法 |
|---------|-----|
| `/wiki` 提示 "not found" | 确保已启用 `claude-obsidian` 插件：`claude plugin list` |
| 关闭 Obsidian 后图谱颜色被重置 | 打开 Graph view → 齿轮 → Color groups → 重新添加一次。之后会永久保留。 |
| Excalidraw 无法加载 | 运行 `bash bin/setup-vault.sh` 以下载 `main.js`（8MB，不在 git 中） |
| 仪表盘没有结果 | 从 Community Plugins 安装 **Dataview** 插件 |
| 会话开始时热缓存未加载 | 检查 hooks：`claude hooks list`；应存在 SessionStart hook |

---

## 跨项目妙招

让任意 Claude Code 项目指向这个仓库。在该项目的 `CLAUDE.md` 中添加：

```markdown
## Wiki Knowledge Base
Path: ~/path/to/claude-obsidian

When you need context not in this project:
1. Read wiki/hot.md first (recent context cache)
2. If not enough, read wiki/index.md
3. If you need domain details, read the relevant wiki page

Do NOT read the wiki for general coding questions.
```

你的行政助理、编码项目和内容工作流都从同一个知识库取用。

---

## 支持

- **GitHub（公开权威仓库）**：[github.com/AgriciDaniel/claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian)
- **Issues**：[github.com/AgriciDaniel/claude-obsidian/issues](https://github.com/AgriciDaniel/claude-obsidian/issues)
- **社区抢先体验（Pro）**：[AI Marketing Hub org](https://github.com/AI-Marketing-Hub) · [Skool community](https://www.skool.com/ai-marketing-hub-pro)

---

*由 [AgriciDaniel](https://github.com/AgriciDaniel) / AI Marketing Hub 构建*
*基于 Andrej Karpathy 的 LLM Wiki 模式*
