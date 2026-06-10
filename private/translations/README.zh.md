---
source: README.md
source_version: 1.9.2
translated_at: 2026-06-10
---

# claude-obsidian：为 Obsidian + Claude Code 打造的自组织 AI 第二大脑

<p align="center">
  <img src="../../wiki/meta/claude-obsidian-gif-cover-16x9.gif" alt="claude-obsidian: persistent compounding wiki vault for Claude Code and Obsidian" width="100%" />
</p>

[![GitHub stars](https://img.shields.io/github/stars/AgriciDaniel/claude-obsidian?style=flat&color=e8734a)](https://github.com/AgriciDaniel/claude-obsidian/stargazers)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](../../LICENSE)
[![Release](https://img.shields.io/github/v/release/AgriciDaniel/claude-obsidian?color=blue)](https://github.com/AgriciDaniel/claude-obsidian/releases/latest)
[![CI](https://github.com/AgriciDaniel/claude-obsidian/actions/workflows/test.yml/badge.svg)](https://github.com/AgriciDaniel/claude-obsidian/actions/workflows/test.yml)
[![Claude Code](https://img.shields.io/badge/Claude_Code-plugin-8B5CF6)](https://code.claude.com/docs/en/discover-plugins)
[![Obsidian](https://img.shields.io/badge/Obsidian-v1.9.10%2B-7c3aed)](https://obsidian.md)
[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-Compatible-blue)](https://agentskills.io)
[![Community](https://img.shields.io/badge/AI%20Marketing%20Hub-Pro%20community-purple)](https://www.skool.com/ai-marketing-hub-pro)
[![Blog Post](https://img.shields.io/badge/Deep_Dive-Blog_Post-22c55e)](https://agricidaniel.com/blog/claude-obsidian-ai-second-brain)

Claude + Obsidian 知识伙伴和自组织 AI 第二大脑。一个持续运行的 AI 笔记助手，会构建并维护一个持久、不断复利增长的 wiki 知识库。你添加的每一份资料都会被整合进去。你提出的每一个问题都会从已经读过的全部内容中提取答案。知识像利息一样不断复利累积。

开源的 Obsidian AI 插件，适用于 AI 笔记、个人知识管理（PKM）、第二大脑工作流，以及作为一个私有的 Notion 替代方案。**15 个 Claude Code skill**、多智能体支持、多写入者安全（v1.7+）、一流的方法论模式（LYT / PARA / Zettelkasten / Generic，v1.8 引入），以及 10 原则思考框架（v1.9）。基于 [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)。

> **获取此 skill 有两种方式。** 选择符合你工作方式的那一种。
>
> - 🌐 **公开开源版**（最新：`v1.9.2`，推荐）：发布在 [Daniel Agrici's GitHub](https://github.com/AgriciDaniel/claude-obsidian) 上的免费、MIT 许可版本。对任何人开放，无需会员资格。包含全部功能：v1.7 Compound Vault、v1.8 方法论模式，以及 v1.9 思考框架和审计加固。
> - ⚡ **AI Marketing Hub Pro**：同样的 MIT 许可核心，外加对开发中功能的最早访问权（在它们落地这里之前）、直接协作，以及 [Pro community](https://www.skool.com/ai-marketing-hub-pro)。Pro 成员从 [AI Marketing Hub](https://github.com/AI-Marketing-Hub) 组织镜像安装（见下方 Option 2 中的替换说明）。

> ✨ **v1.7「Compound Vault」重构**：以 Obsidian CLI 作为默认传输方式、混合检索（contextual prefix + BM25 + cosine rerank，依据 [Anthropic's Sept 2024 research](https://www.anthropic.com/news/contextual-retrieval)）、按文件的咨询式锁定（关闭了一个潜在的多写入者数据损坏漏洞），以及与 [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills) 的底层对齐。完整指南：[docs/compound-vault-guide.md](../../docs/compound-vault-guide.md)。可选的 [DragonScale Memory](../../docs/dragonscale-guide.md) 扩展（日志折叠、确定性页面地址、语义切片 lint、边界优先的 autoresearch）。

---

## 目录

- [What It Does](#what-it-does)
- [Why claude-obsidian?](#why-claude-obsidian)
- [Quick Start](#quick-start)
- [Commands](#commands)
  - [`/wiki`: setup, scaffold, continue](#wiki-setup-scaffold-continue)
  - [`/autoresearch`: autonomous research loop](#autoresearch-autonomous-research-loop)
  - [`/canvas`: visual layer](#canvas-visual-layer)
  - [`/think`: 10-principle thinking loop](#think-10-principle-thinking-loop)
- [Methodology Modes (v1.8+)](#methodology-modes-v18)
- [Vault Use Cases (v1.0+)](#vault-use-cases-v10)
- [Cross-Project Knowledge Base](#cross-project-knowledge-base)
- [What Gets Created](#what-gets-created)
- [Architecture](#architecture)
- [MCP Setup (Optional)](#mcp-setup-optional)
- [Plugins](#plugins)
- [CSS Snippets](#css-snippets-auto-enabled-by-setup-vaultsh)
- [Banner Plugin](#banner-plugin)
- [File Structure](#file-structure)
- [AutoResearch Configuration](#autoresearch-programmd)
- [Seed Vault](#seed-vault)
- [Companion: claude-canvas](#companion-claude-canvas)
- [FAQ](#faq)
- [Requirements](#requirements)
- [Uninstall](#uninstall)
- [Contributing](#contributing)
- [Related Projects](#related-projects)
- [Community](#community)
- [License](#license)

---

## What It Does

### [YouTube Demo](https://www.youtube.com/watch?v=a2hgayvr-H4)

<p align="center">
  <img src="../../wiki/meta/welcome-canvas.gif" alt="claude-obsidian welcome canvas: visual demo of the wiki vault workflow" width="96%" />
</p>

你投放资料。Claude 读取它们，提取实体和概念，更新交叉引用，并把所有内容归档进一个结构化的 Obsidian 知识库。每次 ingest 之后 wiki 都会变得更丰富。

你提问。Claude 读取热缓存（最近上下文），扫描索引，钻取相关页面，然后综合出一个答案。它引用的是具体的 wiki 页面，而不是训练数据。

你做 lint。Claude 会找出孤立页面、失效链接、过时的论断，以及缺失的交叉引用。你的 wiki 无需手动清理就能保持健康。

在每个会话结束时，Claude 都会更新热缓存。下一个会话以完整的最近上下文开始，无需重新概述。

<p align="center">
  <img src="../../wiki/meta/image-example-graph-view.png" alt="Obsidian graph view showing the claude-obsidian knowledge graph with color-coded nodes for concepts, entities, and sources" width="48%" />
  <img src="../../wiki/meta/image-example-wiki-map-view.png" alt="Wiki Map canvas: visual hub linking domain pages, concepts, and entities" width="48%" />
</p>

---

## Why claude-obsidian?

大多数 Obsidian AI 插件都是聊天界面。它们回答关于你现有笔记的问题。claude-obsidian 是一个知识引擎。它会自主地创建、组织、维护并演进你的笔记。

| 能力 | claude-obsidian | Smart Connections | Copilot |
|---|---|---|---|
| **自动组织笔记** | ✅ 创建实体、概念、交叉引用 | ❌ | ❌ |
| **矛盾标记** | ✅ 带来源的 `[!contradiction]` callout | ❌ | ❌ |
| **会话记忆** | ✅ 热缓存在对话之间持久保留 | ❌ | ❌ |
| **知识库维护** | ✅ 8 类 lint（孤立页面、失效链接、缺口） | ❌ | ❌ |
| **自主研究** | ✅ 3 轮网络研究并填补缺口 | ❌ | ❌ |
| **方法论模式** | ✅ LYT / PARA / Zettelkasten / Generic（一流支持） | ❌ | ❌ |
| **思考框架** | ✅ 10 原则循环作为可调用的 skill | ❌ | ❌ |
| **多模型支持** | ✅ Claude、Gemini、Codex、Cursor、Windsurf | ❌ 仅 Claude | ✅ 多种 |
| **可视化 canvas** | ✅ 通过 [claude-canvas](https://github.com/AgriciDaniel/claude-canvas) | ❌ | ❌ |
| **多写入者安全** | ✅ 按文件的咨询式锁（v1.7+） | ❌ | ❌ |
| **带引用的查询** | ✅ 引用具体的 wiki 页面 | ✅ 引用相似笔记 | ✅ 引用笔记 |
| **批量 ingest** | ✅ 多份资料并行 agent 处理 | ❌ | ❌ |
| **开源** | ✅ MIT | ✅ MIT | ⚠️ Freemium |

> 📖 **深入了解：** [I Turned Obsidian Into a Self-Organizing AI Brain](https://agricidaniel.com/blog/claude-obsidian-ai-second-brain)。完整剖析，包含数据可视化、市场背景和工作流演示。

---

## Quick Start

> ℹ️ 下面的命令会从 `AgriciDaniel/claude-obsidian` 安装**公开开源版**（推荐，无需会员）。想要提前体验开发中功能的 **AI Marketing Hub Pro 成员**可以把 `AgriciDaniel/claude-obsidian` 替换为 `AI-Marketing-Hub/claude-obsidian`（Option 2 还需要替换插件 slug；见该选项下的说明）。

### Option 1: Clone as vault（推荐，2 分钟完成完整设置）

```bash
git clone https://github.com/AgriciDaniel/claude-obsidian
cd claude-obsidian
bash bin/setup-vault.sh
```

在 Obsidian 中打开该文件夹：**Manage Vaults → Open folder as vault → 选择 `claude-obsidian/`**。

在同一文件夹中打开 Claude Code。输入 `/wiki`。

> ℹ️ `setup-vault.sh` 会配置 `graph.json`（过滤器 + 颜色）、`app.json`（排除插件目录）和 `appearance.json`（启用 CSS）。在首次打开 Obsidian 前运行它一次。你将获得开箱即用、已完全预配置的图谱视图、配色方案和 wiki 结构。

---

### Option 2: Install as Claude Code plugin

插件安装分两步。先添加 marketplace 目录，再从中安装插件。

> ℹ️ **你要安装哪个版本？**
>
> - **公开版（推荐，无需会员）：** 下面的命令会从 [`AgriciDaniel/claude-obsidian`](https://github.com/AgriciDaniel/claude-obsidian) 安装免费、MIT 许可的版本。无需注册。
> - **AI Marketing Hub Pro 成员？** 想提前体验开发中功能的话，把 `AgriciDaniel/claude-obsidian` 替换为 `AI-Marketing-Hub/claude-obsidian`，并把插件 slug `claude-obsidian@agricidaniel-claude-obsidian` 替换为 `claude-obsidian@ai-marketing-hub-claude-obsidian`。该组织镜像需要一个对 `AI-Marketing-Hub` 组织有访问权限的、已认证的 `gh auth login`（或 GitHub PAT）。如果 `/plugin marketplace add` 返回 404，说明你的账号还不在该组织里。在 [Skool community](https://www.skool.com/ai-marketing-hub-pro) 发私信申请加入。

```bash
# Step 1: add the marketplace
claude plugin marketplace add AgriciDaniel/claude-obsidian

# Step 2: install the plugin
claude plugin install claude-obsidian@agricidaniel-claude-obsidian
```

在任何 Claude Code 会话中：`/wiki`。Claude 会引导你完成知识库设置。

检查是否成功：

```bash
claude plugin list
```

---

### Option 3: Add to an existing vault

把 `WIKI.md` 复制到你的 vault 根目录。粘贴给 Claude：

```
Read WIKI.md in this project. Then:
1. Check if Obsidian is installed. If not, install it.
2. Check if the Local REST API plugin is running on port 27124.
3. Configure the MCP server.
4. Ask me ONE question: "What is this vault for?"
Then scaffold the full wiki structure.
```

---

## Commands

| 你说 | Claude 做什么 |
|---------|------------|
| `/wiki` | 设置检查、scaffold，或从你上次中断处继续 |
| `ingest [file]` | 读取资料，创建 8-15 个 wiki 页面，更新索引和日志 |
| `ingest all of these` | 批量处理多份资料，然后做交叉引用 |
| `what do you know about X?` | 读取索引，钻取相关页面，综合出答案 |
| `/save` | 把当前对话归档为一篇 wiki 笔记 |
| `/save [name]` | 用指定标题保存（跳过命名提问） |
| `/autoresearch [topic]` | 运行自主研究循环：搜索、抓取、综合、归档 |
| `/canvas` | 打开或创建可视化 canvas，列出 zone 和节点 |
| `/canvas add image [path]` | 把一张图片（URL 或本地路径）以自动布局添加到 canvas |
| `/canvas add text [content]` | 把一张 markdown 文本卡片添加到 canvas |
| `/canvas add pdf [path]` | 把一个 PDF 文档添加为渲染的预览节点 |
| `/canvas add note [page]` | 把一个 wiki 页面以链接卡片的形式钉到 canvas 上 |
| `/canvas zone [name]` | 添加一个新的带标签的 zone 来组织可视化内容 |
| `/canvas from banana` | 把最近生成的图片捕获到 canvas 上 |
| `/think [problem]` | 对一个非平凡问题应用 10 原则思考循环 |
| `lint the wiki` | 健康检查：孤立页面、失效链接、缺口、建议 |
| `update hot cache` | 用最新的上下文摘要刷新 hot.md |

> ✨ **想要更多？** [claude-canvas](https://github.com/AgriciDaniel/claude-canvas) 增加了 12 个模板、6 种布局算法、AI 图像生成、演示文稿，以及完整的 canvas 编排。两个都装上，它们互为补充。

### `/wiki`: setup, scaffold, continue

首次运行的设置流程包括：

1. 检查 Obsidian 是否已安装
2. 检查 Local REST API plugin（如果需要 MCP 传输方式）
3. 提问「What is this vault for?」（仅一个问题，驱动 scaffold）
4. 按所选的 [Methodology Mode](#methodology-modes-v18) 和 [Vault Use Case](#vault-use-cases-v10) 进行 scaffold
5. 初始化 `hot.md`、`index.md`、`log.md`、`wiki/meta/dashboard.base`
6. 建议第一次 ingest

在后续运行中，`/wiki` 会从你上次中断处继续。它会检查知识库健康状况、浮现过时的论断，并从 `hot.md` 展示最近的活动。

### `/autoresearch`: autonomous research loop

可配置程序位于 [`skills/autoresearch/references/program.md`](../../skills/autoresearch/references/program.md)：

- 最大轮数（默认 3）
- 每个会话的最大页面数（默认 15）
- 来源偏好规则（学术、官方文档、新闻）
- 置信度评分 + 域名约束

该循环：

1. **第 1 轮，广泛搜索**：拆解为 3-5 个角度，每个角度运行 2-3 个查询，每个角度抓取前 2-3 个结果
2. **第 2 轮，填补缺口**：针对矛盾和缺失部分进行定向搜索
3. **第 3 轮，综合检查**（可选）：如果仍有重大缺口，再做一次
4. **归档**：综合页 + 来源页 + 实体页 + 概念页，全部交叉引用

URL 校验 + 内容净化按 [`skills/autoresearch/SKILL.md`](../../skills/autoresearch/SKILL.md) 中的 `## Web egress hygiene (v1.8.2+)` 策略执行：拒绝 `file://` / `javascript:` / RFC1918 主机，剥离 `<script>` 和 wikilink 注入尝试，把抓取的正文上限设为 50KB。

### `/canvas`: visual layer

把图片、PDF、笔记和 AI 生成的图片添加到 Obsidian canvas。通过 zone 管理来分组。自动布局会在不重叠的情况下放置节点。

```
/canvas                       # open or create the canvas
/canvas add image <path>      # add an image with auto-layout
/canvas add pdf <path>        # render PDF as preview node
/canvas add note <wiki-page>  # pin a wiki page as a linked card
/canvas zone <name>           # add a labeled zone
/canvas from banana           # capture recent banana-generated images
```

符合 JSON Canvas 1.0 规范（[`skills/canvas/references/canvas-spec.md`](../../skills/canvas/references/canvas-spec.md)）。完整编排（12 个模板、6 种布局算法、演示文稿）见配套的 [claude-canvas](https://github.com/AgriciDaniel/claude-canvas)。

### `/think`: 10-principle thinking loop

对任何非平凡问题（架构决策、审计、复盘、含糊不清的用户请求）应用 OBSERVE-OBSERVE-LISTEN-THINK-CONNECT-CONNECT-FEEL-ACCEPT-CREATE-GROW 框架。

```
/think <problem statement>
```

该框架带领 Claude 走过 10 个阶段，每个阶段都有相应提示。当问题的新颖度 + 不可逆性足以证明这种纪律性是值得的时候使用它。完整框架见 [`skills/think/SKILL.md`](../../skills/think/SKILL.md)。其他每个 skill 都有一个「How to think」附录，把该框架映射到它自己的具体工作上。[v1.8.0 pre-push audit](../../docs/audits/v1.8.0-pre-push-audit-2026-05-18.md) 就以该框架作为其方法论主轴。

---

## Methodology Modes (v1.8+)

四种组织哲学，通过 `bash bin/setup-mode.sh` 选择性启用。`wiki-mode` skill（v1.8+）读取 `.vault-meta/mode.json` 并据此为新页面路由。默认是 `generic`（v1.7 行为，不强加任何观点）。

| 模式 | 哲学 | 归档约定 |
|------|-----------|-------------------|
| **Generic**（默认） | 不强加观点。保留 v1.7 行为。 | `wiki/sources/`、`wiki/entities/`、`wiki/concepts/`、`wiki/sessions/` |
| **LYT**（Linking Your Thinking） | 笔记之间互链，文件夹不承担组织职责。MOC 是导航的基本单元。 | `wiki/mocs/<topic>-moc.md` + `wiki/notes/<atomic-note>.md` |
| **PARA**（Tiago Forte） | 按可执行性组织（Projects、Areas、Resources、Archives）。 | `wiki/projects/`、`wiki/areas/`、`wiki/resources/`、`wiki/archives/` |
| **Zettelkasten**（Luhmann 卡片盒） | 原子笔记、唯一 ID、密集的双向链接、无文件夹。 | `wiki/<YYYYMMDDHHMMSSffffff>-<slug>.md`（扁平、带时间戳） |

切换模式**不会**自动迁移已有文件。完整指南：[`docs/methodology-modes-guide.md`](../../docs/methodology-modes-guide.md)。

---

## Vault Use Cases (v1.0+)

它们描述你的 vault 是**用来做什么**的。它们与 Methodology Modes（描述它是**如何**组织的）组合使用。

| 用例 | 何时使用 |
|----------|-------------|
| **A: Website** | 站点地图、内容审计、SEO wiki |
| **B: GitHub** | 代码库地图、架构 wiki |
| **C: Business** | 项目 wiki、竞争情报 |
| **D: Personal** | 第二大脑、目标、日记综合 |
| **E: Research** | 论文、概念、论文（thesis） |
| **F: Book/Course** | 章节追踪、课程笔记 |

用例可以组合。一个用 PARA 组织的 Business + Research vault 是有效的组合。

---

## Cross-Project Knowledge Base

让任何 Claude Code 项目指向这个 vault。在该项目的 `CLAUDE.md` 中添加：

```markdown
## Wiki Knowledge Base
Path: ~/path/to/vault

When you need context not already in this project:
1. Read wiki/hot.md first (recent context cache)
2. If not enough, read wiki/index.md
3. If you need domain details, read the relevant domain sub-index
4. Only then drill into specific wiki pages

Do NOT read the wiki for general coding questions or tasks unrelated to [domain].
```

你的行政助理、编码项目和内容工作流都从同一个知识库中取用。

---

## What Gets Created

一次典型的 scaffold 会创建：

- 针对你所选用例 + 方法论模式的文件夹结构
- `wiki/index.md`：主目录
- `wiki/log.md`：仅追加的操作日志
- `wiki/hot.md`：最近上下文缓存
- `wiki/overview.md`：执行摘要
- `wiki/meta/dashboard.base`：Bases 仪表盘（主用，Obsidian 原生）
- `wiki/meta/dashboard.md`：传统 Dataview 仪表盘（可选后备）
- `_templates/`：每种笔记类型的 Obsidian Templater 模板
- `.obsidian/snippets/vault-colors.css`：按颜色编码的文件浏览器
- Vault `CLAUDE.md`：自动加载的项目说明

---

## Architecture

三张图解释该插件实质性的设计选择。

### Vault flow

资料落入 `.raw/`。`/wiki-ingest` agent 读取每份资料，提取实体和概念，把它们归档到合适的 `wiki/` 子文件夹（按当前激活的方法论模式），并更新索引、日志和热缓存。查询按 hot → index → pages 的顺序读取，以保持 token 成本较低。

<p align="center">
  <img src="../../assets/diagrams/vault-flow.svg" alt="Architecture diagram: sources flow into the wiki-ingest agent, which produces entity, concept, and source pages. The index and hot cache are updated. The wiki-query interface reads the cache, index, and pages to synthesize cited answers." width="100%" />
</p>

### Multi-writer safety (v1.7+)

如果用户批量处理多份资料，并行的 ingest 子 agent 可能会指向同一个 wiki 页面。`scripts/wiki-lock.sh` 提供按文件的咨询式锁：一个写入者获取锁，另一个等待并在下一轮重试。PostToolUse 自动提交钩子在暂存前会检查锁列表，在写入进行中时推迟提交。

<p align="center">
  <img src="../../assets/diagrams/multi-writer-locking.svg" alt="Architecture diagram: two parallel writers attempt to acquire a lock on the same wiki page via wiki-lock.sh. One writer is granted, writes the page, and releases the lock. The other writer logs the skip and retries on the next pass. No corruption, no half-written pages." width="100%" />
</p>

### Hybrid retrieval (v1.7+, opt-in)

`/wiki-retrieve` skill 提供一个三层检索流水线，基于 [Anthropic's Sept 2024 contextual retrieval research](https://www.anthropic.com/news/contextual-retrieval)。BM25 是始终开启的稀疏层。contextual-prefix 层受同意门控（`--allow-egress`），供那些愿意把页面正文发送到 Anthropic API 以生成前缀的用户使用。Cosine rerank 默认使用本地 ollama 模型。v1.7 中的 50 查询基准测试测得相对 v1.6 基线 top-1 准确率提升 32 个百分点、错误减少 41%。

<p align="center">
  <img src="../../assets/diagrams/hybrid-retrieval.svg" alt="Architecture diagram: user query feeds both BM25 sparse search and an optional contextual-prefix Anthropic API call. Both feed a cosine rerank via local ollama embeddings. The output is a ranked list of candidates with --explain traceability for every score." width="100%" />
</p>

> ℹ️ 用 `bash bin/setup-retrieve.sh` 来配置该流水线。它会构建 BM25 索引、提示 egress 同意，并校验 ollama 连接。该流水线会优雅降级：如果某一层不可用，其余各层仍会返回有用的结果。

---

## MCP Setup (Optional)

MCP 让 Claude 无需复制粘贴就能直接读写 vault 笔记。

**Option A（基于 REST API）：**

1. 在 Obsidian 中安装 Local REST API plugin
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

**Option B（基于文件系统，无需插件）：**

```bash
claude mcp add-json obsidian-vault '{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@bitbonsai/mcpvault@latest", "/path/to/your/vault"]
}' --scope user
```

> ℹ️ 两种传输方式都会被 `scripts/detect-transport.sh` 自动检测。结果落在 `.vault-meta/transport.json`。要固定一个手动选择，编辑该文件并设置 `"manual_override": true`（v1.8.2+ 会遵从它）。

---

## Plugins

### Core Plugins（Obsidian 内置，无需安装）

| 插件 | 用途 |
|--------|---------|
| **Bases** | 为 `wiki/meta/dashboard.base` 提供动力：原生数据库视图。自 Obsidian v1.9.10（2025 年 8 月）起可用。在主仪表盘上取代 Dataview。 |
| **Properties** | 可视化 frontmatter 编辑器 |
| **Backlinks**、**Outline**、**Graph view** | 标准导航 |

### 预装的社区插件（随此 vault 一起提供）

在 **Settings → Community Plugins → enable** 中启用：

| 插件 | 用途 | 备注 |
|--------|---------|-------|
| **Calendar** | 右侧栏日历，带字数统计 + 任务圆点 | 预装 |
| **Thino** | 快速备忘捕获面板 | 预装 |
| **Excalidraw** | 手绘画布，标注图片 | 预装* |
| **Banners** | 通过 `banner:` frontmatter 实现的 Notion 风格头图 | 预装 |

\* Excalidraw 的 `main.js`（8MB）由 `setup-vault.sh` 自动下载。它不在 git 中跟踪。

### 也请从社区插件安装（未预装）

| 插件 | 用途 |
|--------|---------|
| **Templater** | 从 `_templates/` 自动填充 frontmatter |
| **Obsidian Git** | 每 15 分钟自动提交 vault |
| **Dataview** *（可选，传统）* | 仅旧版 `wiki/meta/dashboard.md` 查询需要。主仪表盘现在使用 Bases。 |

另外安装 **[Obsidian Web Clipper](https://obsidian.md/clipper)** 浏览器扩展。一键把网页发送到 `.raw/`。

---

## CSS Snippets（由 setup-vault.sh 自动启用）

三个代码片段随 vault 提供并会被自动启用：

| 片段 | 效果 |
|---------|--------|
| `vault-colors` | 在文件浏览器中按类型对 `wiki/` 文件夹做颜色编码（蓝 = concepts，绿 = sources，紫 = entities） |
| `ITS-Dataview-Cards` | 把 Dataview `TABLE` 查询变成可视化卡片网格：用 ` ```dataviewjs ` 配合 `.cards` 类 |
| `ITS-Image-Adjustments` | 笔记中细粒度的图片尺寸控制：在任意图片嵌入后追加 `\|100` |

---

## Banner Plugin

添加到任意 wiki 页面的 frontmatter：

```yaml
banner: "_attachments/images/your-image.png"
banner_icon: "🧠"
```

该页面会在 Obsidian 中渲染一张全宽头图。对枢纽页面和概览页效果很好。

---

## File Structure

```
claude-obsidian/
├── .claude-plugin/
│   ├── plugin.json              # manifest
│   └── marketplace.json         # distribution
├── skills/                       # 15 Claude Code skills (v1.9.2)
│   ├── wiki/                    # orchestrator + references
│   ├── wiki-ingest/             # source ingestion
│   ├── wiki-query/              # answer questions from the vault
│   ├── wiki-lint/               # vault health check
│   ├── wiki-cli/                # Obsidian CLI transport (v1.7+)
│   ├── wiki-retrieve/           # hybrid retrieval (v1.7+, opt-in)
│   ├── wiki-mode/               # methodology modes router (v1.8+)
│   ├── wiki-fold/               # log rollup (DragonScale opt-in)
│   ├── save/                    # /save: file conversations to wiki
│   ├── autoresearch/            # autonomous research loop
│   ├── canvas/                  # visual layer (images, PDFs, notes)
│   ├── defuddle/                # web extraction wrapper
│   ├── obsidian-bases/          # Bases schema reference
│   ├── obsidian-markdown/       # OFM syntax reference
│   └── think/                   # 10-principle thinking framework (v1.9+)
├── agents/
│   ├── verifier.md              # pre-commit audit agent (v1.7.1+)
│   ├── wiki-ingest.md           # parallel batch ingestion agent
│   └── wiki-lint.md             # health check agent
├── commands/                     # slash command entry points
├── hooks/
│   └── hooks.json               # SessionStart + Stop + PostToolUse hooks
├── scripts/                      # 12 helper scripts (transport, locking, retrieval, etc.)
├── tests/                        # 9 hermetic test suites (~1240 assertions, make test)
├── bin/                          # 5 setup scripts (setup-vault, setup-retrieve, setup-mode, etc.)
├── _templates/                   # Obsidian Templater templates
├── wiki/                         # seeded vault content (demo)
│   ├── canvases/                # welcome.canvas + main.canvas
│   ├── concepts/                # seeded: LLM Wiki Pattern, Hot Cache, Compounding Knowledge
│   ├── entities/                # seeded: Andrej Karpathy
│   ├── sources/                 # populated by your first ingest
│   └── meta/
│       ├── dashboard.base       # Bases dashboard (primary)
│       └── dashboard.md         # Legacy Dataview dashboard (optional)
├── docs/                         # guides + audits + release notes
├── .raw/                         # source documents (hidden in Obsidian)
├── .obsidian/snippets/           # vault-colors.css (3-color scheme)
├── WIKI.md                       # full schema reference
├── CLAUDE.md                     # project instructions
└── README.md                     # this file
```

---

## AutoResearch: program.md

`/autoresearch` 命令是可配置的。编辑 [`skills/autoresearch/references/program.md`](../../skills/autoresearch/references/program.md) 来控制：

- 偏好哪些来源（学术、官方文档、新闻）
- 置信度评分规则
- 每个会话的最大轮数和最大页面数
- 特定领域的约束

默认程序适用于一般研究。针对你的领域可以覆盖它。一名医学研究者会加上「prefer PubMed」。一名商业分析师会加上「focus on market data and filings」。

---

## Seed Vault

本仓库附带一个已初始化（seeded）的 vault。在 Obsidian 中打开它，你会看到：

- `wiki/concepts/`：LLM Wiki Pattern、Hot Cache、Compounding Knowledge
- `wiki/entities/`：Andrej Karpathy
- `wiki/sources/`：在你第一次 ingest 之前为空
- `wiki/meta/dashboard.base`：Bases 仪表盘（在任何 Obsidian v1.9.10+ 中可用）
- `wiki/meta/dashboard.md`：传统 Dataview 仪表盘（可选后备）

图谱视图会展示一个由 5 个页面相连的集群。这就是一次 ingest 之后 wiki 的样子。添加更多资料，它就会从这里继续生长。

<p align="center">
  <img src="../../wiki/meta/wiki-graph-grow.gif" alt="Animated GIF: claude-obsidian knowledge graph growing from a few seeded pages to a dense web of cross-referenced concepts after multiple ingests" width="48%" />
  <img src="../../wiki/meta/workflow-loop.gif" alt="Animated GIF: claude-obsidian workflow loop showing ingest, query, lint, save, and hot-cache refresh cycle" width="48%" />
</p>

---

## Companion: claude-canvas

对于可视化层，[claude-canvas](https://github.com/AgriciDaniel/claude-canvas) 增加了 AI 编排的 canvas 创建：知识图谱、演示文稿、流程图、情绪板，配有 12 个模板和 6 种布局算法。会自动检测 claude-obsidian vault。

```bash
claude plugin install AgriciDaniel/claude-canvas
```

---

## FAQ

**最好的 AI 第二大脑应用是哪个？**
最好的 AI 第二大脑会让你的数据始终归你所有。claude-obsidian 把所有内容存为你拥有的纯 Markdown 文件（无数据库、无锁定、无订阅），并让 Claude 读取、链接并把它们组织进一个相连的知识图谱。它免费且开源（MIT）。

**如何用 AI 构建第二大脑？**
把任意资料投放进 vault。Claude 读取它，提取实体和概念，把它们与你已有的内容链接起来，并归档进一个结构化的 Obsidian vault。你提问；它从读过的全部内容中作答并引用页面。知识库会随着每个会话变得更丰富、更紧密相连。

**如何把 Claude 接入 Obsidian 作为第二大脑？**
两行：`git clone https://github.com/AgriciDaniel/claude-obsidian`，然后 `cd claude-obsidian && bash bin/setup-vault.sh`。把该文件夹作为 Obsidian vault 打开，在同一文件夹中打开 Claude Code，输入 `/wiki`。完整步骤见 [Quick Start](#quick-start)。

**有没有适合做私有、AI 驱动知识库的好用 Notion 替代品？**
有。claude-obsidian 是一个开源、本地优先的替代方案：你的笔记是放在你自己磁盘上的纯 Markdown，而不是托管数据库，并且由 AI 替你组织。没有厂商锁定，也没有月费。

**这个会跨设备自动同步吗？**
本身不会。vault 就是一个普通的 Markdown 文件夹。搭配 Obsidian Sync、Obsidian Git，或任意文件同步工具（Syncthing、iCloud、Dropbox）来实现跨设备同步。

**多人能否安全地编辑同一个 vault？**
可以（v1.7+）。通过 [`scripts/wiki-lock.sh`](../../scripts/wiki-lock.sh) 的按文件咨询式锁定防止并发写入损坏页面。并行的 ingest 子 agent 在写入前获取锁。过期的锁会在 60 秒后自我回收。

**`hot.md` 和 `index.md` 有什么区别？**
`hot.md` 是最近上下文缓存（约 500 词，每个会话刷新）。`index.md` 是 vault 中每个页面的主目录。Claude 先读 `hot.md`，再读 `index.md`，然后钻取到具体页面。这种两层设计让重复查询保持较低的 token 成本。

**不用 Claude Code 能用这个吗？**
这些 skill 兼容 Agent Skills（对 OpenAI Codex CLI、Cursor、Windsurf、Gemini CLI、Goose 有实验性支持）。目前只在 Claude Code 上做了生产验证。跨宿主的安装路径遵循各宿主的约定，但 skill 发现方式可能有所不同。

**如何从 Dataview 迁移到 Bases？**
两者并排提供。`wiki/meta/dashboard.base` 是主用；`wiki/meta/dashboard.md` 是传统 Dataview 后备。在 Obsidian 中选一个，另一个无害。Bases 需要 Obsidian v1.9.10+（2025 年 8 月）。

**Methodology Modes（LYT/PARA/Zettelkasten）和 Vault Use Cases（Website/GitHub/Business）有什么区别？**
Methodology Modes（v1.8+）控制页面**如何**组织：文件夹结构 + 文件名约定。Vault Use Cases（v1.0+）描述 vault 是**用来做什么**的：内容类型。它们可组合。一个使用 PARA 方法论的「Business」vault 是有效的配置。

**这个会把我的笔记发送给 Anthropic 吗？**
默认不会。可选的 `/wiki-retrieve` skill 的 API egress（`contextual-prefix.py`）被 `--allow-egress` 同意标志门控。没有该标志，检索完全在本地（BM25 + 可选的 ollama rerank）。`/autoresearch` 中的 Web egress 遵循同样的选择性启用原则。

**公开版和 AI Marketing Hub Pro 有什么区别？**
两者共享 [`AgriciDaniel/claude-obsidian`](https://github.com/AgriciDaniel/claude-obsidian) 上同样的 MIT 许可核心，这是面向所有人的推荐安装方式。AI Marketing Hub Pro 成员能最早访问开发中功能（在它们落地这里之前），外加直接协作和社区。核心中没有任何仅付费才有的功能。

**什么是 DragonScale Memory？**
一个可选、选择性启用的扩展（`bash bin/setup-dragonscale.sh`），它增加四种记忆机制：日志折叠（对过往条目的汇总）、确定性页面地址（基于计数器的唯一 ID）、语义切片 lint（通过 ollama 做块边界校验），以及边界优先的 autoresearch（先研究 vault 的「前沿」）。正常使用不需要它。完整指南：[`docs/dragonscale-guide.md`](../../docs/dragonscale-guide.md)。

---

## Requirements

| 组件 | 最低要求 | 备注 |
|-----------|---------|-------|
| Claude Code | latest | https://claude.com/claude-code |
| Obsidian | v1.9.10+（用于 Bases） | https://obsidian.md。v1.6+ 配合 Dataview 后备可用。 |
| Python | 3.10+ | 用于可选的检索流水线和测试套件 |
| Bash | 4.0+（或 zsh） | 用于 setup 脚本 |
| Git | any | 用于通过 Obsidian Git 插件做 vault 自动提交 |

**可选：**

- **ollama**（用于 `/wiki-retrieve` 中的本地 rerank）
- **defuddle-cli**（用于 `/defuddle` 中的干净网页提取）
- **Anthropic API key**（用于 `/wiki-retrieve` 的 contextual prefix 层，通过 `--allow-egress` 选择性启用）
- **Local REST API plugin**（用于 REST-API MCP 传输方式）

---

## Uninstall

插件安装方式：

```bash
claude plugin uninstall claude-obsidian@agricidaniel-claude-obsidian
claude plugin marketplace remove AgriciDaniel/claude-obsidian
```

Clone 安装方式（删除该文件夹）：

```bash
rm -rf /path/to/claude-obsidian
```

你的 vault 内容（在 `wiki/` 下）是纯 Markdown，卸载后依然存在。要在不卸载的情况下清除运行时状态，从仓库根目录运行 `make clean-test-state`。

---

## Contributing

欢迎 PR。先读这些：

- [`CONTRIBUTING.md`](../../CONTRIBUTING.md)：工作流、six-cut 自审清单、提交约定、hermetic 测试要求
- [`CODE_OF_CONDUCT.md`](../../CODE_OF_CONDUCT.md)：Contributor Covenant v2.1
- [`SECURITY.md`](../../SECURITY.md)：负责任的安全披露策略
- [`CHANGELOG.md`](../../CHANGELOG.md)：版本历史（最新：v1.9.2）

Issue + PR 模板见 [`.github/`](../../.github/)。CI 在每个 PR 上运行 `make test` + SKILL.md frontmatter 校验 + 插件 manifest JSON 有效性检查。位于 [`agents/verifier.md`](../../agents/verifier.md) 的 pre-commit verifier agent 对暂存的 diff 应用 six-cut + agent kernel。

---

## Related Projects

- 🎨 [**claude-canvas**](https://github.com/AgriciDaniel/claude-canvas)：可视化 canvas 编排（12 个模板、6 种布局算法、AI 图像生成）。本插件的配套项目。
- 📊 [**claude-ads**](https://github.com/AgriciDaniel/claude-ads)：多平台付费广告审计（覆盖 Google、Meta、LinkedIn、TikTok、Microsoft、Apple、Amazon Ads 的 250+ 项检查）。
- 🔍 [**claude-seo**](https://github.com/AgriciDaniel/claude-seo)：技术 SEO + GEO 审计套件。
- 🧠 [**best-practices**](https://github.com/AgriciDaniel/best-practices)：可组合的工程内核。`agents/verifier.md` 所执行的 six-cut + agent kernel 的来源。

---

## Community

- 📝 [**Blog post**](https://agricidaniel.com/blog/claude-obsidian-ai-second-brain)：深入剖析，含竞品分析、数据图表和工作流演示
- 💬 [**AI Marketing Hub**](https://www.skool.com/ai-marketing-hub)：2,800+ 成员，免费社区
- ⚡ [**AI Marketing Hub Pro**](https://www.skool.com/ai-marketing-hub-pro)：最早访问开发中功能和直接协作
- 🎬 [**YouTube**](https://www.youtube.com/@AgriciDaniel)：教程和演示
- 🔧 [**All open-source tools**](https://github.com/AgriciDaniel)：claude-seo、claude-ads、claude-blog 等等

---

## License

MIT License。完整文本见 [LICENSE](../../LICENSE)。可免费用于个人和商业用途。欢迎署名但非必需。

---

## Star History

<a href="https://star-history.com/#AgriciDaniel/claude-obsidian&Date">
  <img src="https://api.star-history.com/svg?repos=AgriciDaniel/claude-obsidian&type=Date" alt="Star history chart for AgriciDaniel/claude-obsidian on GitHub" width="640" />
</a>

---

*基于 [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)。由 [Agrici Daniel](https://agricidaniel.com/about) 构建。复利增长的知识，是一个善于思考的人能养成的最高杠杆习惯。*
