---
source: WIKI.md
source_version: 1.9.2
translated_at: 2026-06-10
---

# WIKI.md — LLM Wiki 模式说明

> 如果你正在使用 claude-obsidian 插件，这里描述的一切都由各个 skill 自动处理。
> 本文件是参考文档。阅读它以理解系统的工作原理。
> 基于 Andrej Karpathy 的 LLM Wiki 模式。

---

## 这是什么

你正在维护一个持久、不断复利增长的 wiki，它位于一个 Obsidian vault 之中。你不只是回答问题，而是构建并维护一个结构化的知识库；每加入一个来源、每回答一个问题，它都会变得更丰富。人类负责筛选来源、提出问题。所有的写作、交叉引用、归档和维护都由你完成。

wiki 才是产品。聊天只是接口。

它与 RAG 的关键区别在于：wiki 是一个持久的产物。交叉引用早已就位。矛盾之处已被标记。综合内容已经反映了读过的所有材料。知识像利息一样复利增长。

---

## 0 — 引导：首次运行设置

在任何新项目中首次运行时，按顺序执行以下步骤。已完成的步骤可跳过。

### 0.1 检查 Obsidian 是否已安装

```bash
# Linux: check flatpak first, then PATH
flatpak list 2>/dev/null | grep -i obsidian && echo "FOUND via flatpak" || \
which obsidian 2>/dev/null && echo "FOUND in PATH" || echo "NOT FOUND"

# macOS
ls /Applications/Obsidian.app 2>/dev/null && echo "FOUND" || echo "NOT FOUND"

# Windows (PowerShell)
Test-Path "$env:LOCALAPPDATA\Obsidian" && echo "FOUND" || echo "NOT FOUND"
```

如果尚未安装：

```bash
# Linux (Flatpak)
flatpak install flathub md.obsidian.Obsidian

# macOS (Homebrew)
brew install --cask obsidian

# Windows (winget)
winget install Obsidian.Obsidian

# All platforms: https://obsidian.md/download
```

安装之后：Obsidian > Manage Vaults > Open Folder as Vault > 选择 vault 目录。

如果没有可用的包管理器，告诉用户：“从 https://obsidian.md 下载 Obsidian —— 安装它，创建一个 vault，并把路径告诉我。”

### 0.2 Vault 位置

询问 vault 路径，或使用默认值：

```
VAULT_PATH=~/Documents/Obsidian Vault
```

验证：`ls "$VAULT_PATH/.obsidian" 2>/dev/null`

### 0.3 安装 Local REST API 插件

引导用户操作（你无法以编程方式完成）：

1. Obsidian > Settings > Community Plugins > 关闭 Restricted Mode
2. Browse > 搜索 "Local REST API" > Install > Enable
3. Settings > Local REST API > 复制 API key
4. 插件运行在 `https://127.0.0.1:27124`（自签名证书）

测试：`curl -sk -H "Authorization: Bearer <KEY>" https://127.0.0.1:27124/`

### 0.4 配置 MCP 服务器

**方案 A：mcp-obsidian（基于 REST API，最流行）**

```bash
claude mcp add-json obsidian-vault '{
  "type": "stdio",
  "command": "uvx",
  "args": ["mcp-obsidian"],
  "env": {
    "OBSIDIAN_API_KEY": "<KEY>",
    "OBSIDIAN_HOST": "127.0.0.1",
    "OBSIDIAN_PORT": "27124",
    "NODE_TLS_REJECT_UNAUTHORIZED": "0"
  }
}' --scope user
```

**方案 B：MCPVault（基于文件系统，无需插件）**

```bash
claude mcp add-json obsidian-vault '{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@bitbonsai/mcpvault@latest", "<VAULT_PATH>"]
}' --scope user
```

**方案 C：通过 curl 直连 REST API** —— 总是有效，无需 MCP。见第 11 节。

使用 `--scope user`，让这个 vault 在所有项目中都可用。

**验证：**

```bash
claude mcp list               # confirm server appears
claude mcp get obsidian-vault # confirm path is correct
```

在 Claude Code 会话中，输入 `/mcp` 检查连接状态。

### 0.5 推荐插件

通过 Settings > Community Plugins > Browse 安装：

| 插件 | 用途 |
|--------|-----|
| **Dataview** | 把 vault 当作数据库查询。为各种仪表盘提供支持。 |
| **Templater** | 创建笔记时自动填充 frontmatter。 |
| **Obsidian Git** | 每 15 分钟自动提交。防止数据丢失。 |
| **Iconize** | 可视化的文件夹图标。 |
| **Minimal Theme** | 适合密集信息展示的最佳深色主题。 |

可选：Smart Connections（语义搜索）、QuickAdd（宏）、Folder Notes（可点击的文件夹）。

另外安装 **Obsidian Web Clipper** 浏览器扩展。它能把网页文章转换为 markdown，一键发送到 `.raw/`。支持 Chrome、Firefox 和 Safari。

---

## 1 — 架构

```
vault/
├── .raw/                   # Layer 1: immutable source documents
│   ├── articles/
│   ├── transcripts/
│   ├── screenshots/
│   ├── data/
│   └── assets/
│
├── wiki/                   # Layer 2: LLM-generated knowledge base
│   ├── index.md            # master catalog of all wiki pages
│   ├── log.md              # chronological record of all operations
│   ├── hot.md              # hot cache: recent context summary (~500 words)
│   ├── overview.md         # executive summary of the entire wiki
│   ├── sources/            # one summary page per raw source
│   ├── entities/           # people, orgs, products, repos
│   │   └── _index.md
│   ├── concepts/           # ideas, patterns, frameworks
│   │   └── _index.md
│   ├── domains/            # top-level topic areas
│   │   └── _index.md
│   ├── comparisons/        # side-by-side analyses
│   ├── questions/          # filed answers to user queries
│   └── meta/               # dashboards, lint reports, conventions
│
├── _templates/             # Templater templates
├── _attachments/           # images and PDFs referenced by wiki pages
│
├── WIKI.md                 # Layer 3: this file
└── .obsidian/              # Obsidian config (auto-managed)
```

### 规则

- `.raw/` 是只读的。永不修改源文件。
- `wiki/` 归你所有。可自由创建、更新、重命名、删除。
- 每个 wiki 页面都有 frontmatter。没有例外。
- 优先用 wikilink 而非路径。用 `[[Page Name]]`，不要用 `[text](path/to/file.md)`。
- 原子化笔记。一页一个概念。如果一页覆盖了两件事，就拆开。
- 更新，不要重复。如果页面已存在，就更新它。

---

## 2 — 热缓存（Hot Cache）

`wiki/hot.md` 是对最近上下文的一段约 500 词的摘要。它的存在是为了让指向这个 vault 的其他项目无需爬取整个 wiki 就能获取最近的上下文。

在每次 ingest 之后、任何重要的查询交流之后、以及每次会话结束时，更新 hot.md。

格式：

```markdown
---
type: meta
title: "Hot Cache"
updated: 2026-04-07T14:30:00
---

# Recent Context

## Last Updated
2026-04-07 — Ingested 3 new YouTube transcripts

## Key Recent Facts
- [Most important recent takeaway]
- [Second most important]

## Recent Changes
- Created: [[New Page 1]], [[New Page 2]]
- Updated: [[Existing Page]] (added section on X)
- Flagged: Contradiction between [[Page A]] and [[Page B]] on topic Y

## Active Threads
- User is currently researching [topic]
- Open question: [thing still being investigated]
```

保持在 500 词以内。它是缓存，不是日记。每次都完整覆盖重写。

---

## 3 — Frontmatter 模式

每个 wiki 页面都以扁平的 YAML frontmatter 开头。不要嵌套对象。Obsidian 的 Properties UI 不支持嵌套。

### 通用字段（每个页面都有）：

```yaml
---
type: <source|entity|concept|domain|comparison|question|overview|meta>
title: "Human-Readable Title"
created: 2026-04-07
updated: 2026-04-07
tags:
  - <domain-tag>
  - <type-tag>
status: <seed|developing|mature|evergreen>
related:
  - "[[Other Page]]"
sources:
  - "[[.raw/articles/source-file.md]]"
---
```

### 按类型追加的字段：

**source**：`source_type`、`author`、`date_published`、`url`、`confidence`（high|medium|low）、`key_claims`（列表）

**entity**：`entity_type`（person|organization|product|repository|place）、`role`、`first_mentioned`

**concept**：`complexity`（basic|intermediate|advanced）、`domain`、`aliases`（列表）

**comparison**：`subjects`（wikilink 列表）、`dimensions`（列表）、`verdict`（一行结论）

**question**：`question`（原始查询）、`answer_quality`（draft|solid|definitive）

---

## 4 — 操作

### 4.1 SCAFFOLD —— 首次运行的结构搭建

触发条件：用户描述这个 vault 用来做什么。

1. 确定 wiki 模式（见下方模式表，以及第 4.1a 节的完整模式细节）。
2. 问一个问题：“这个 vault 是用来做什么的？”
3. 在 `wiki/` 下创建完整的文件夹结构。
4. 为每个领域（domain）创建一个领域页面 + `_index.md` 子索引。
5. 创建 `wiki/overview.md`、`wiki/index.md`、`wiki/log.md`、`wiki/hot.md`。
6. 创建 `_templates/`，为每种笔记类型准备模板。
7. 应用视觉定制（第 7 节）。创建 `.obsidian/snippets/vault-colors.css`。
8. 创建 vault 的 CLAUDE.md（模板见第 4.1b 节）。
9. 初始化 git（第 8 节）。
10. 展示结构并询问：“开始之前还想调整些什么吗？”

**模式选择：**

| 用户说 | 最佳模式 |
|-----------|----------|
| "my website"、"sitemap"、"content audit" | A：网站 |
| "my repo"、"codebase map"、"architecture wiki" | B：GitHub |
| "my business"、"project wiki"、"competitive intel" | C：商业 |
| "second brain"、"goals"、"journal"、"my life" | D：个人 |
| "research topic"、"papers"、"deep dive" | E：研究 |
| "book I'm reading"、"course notes"、"chapter tracker" | F：书籍 / 课程 |

你可以组合多种模式。“GitHub repo + research on the AI approach” 会用到模式 B 的文件夹加上模式 E 的 papers/ 文件夹。

### 4.1a —— 六种 Wiki 模式

**模式 A：网站 / 站点地图**

```
vault/
├── .raw/              # crawl exports, analytics, GSC data
├── wiki/
│   ├── pages/         # one note per URL
│   ├── structure/     # site architecture, nav hierarchy
│   ├── audits/        # content gaps, redirect needs
│   ├── keywords/      # keyword clusters, target page assignments
│   └── entities/      # brand, authors, topic hubs
```

pages/ 的 frontmatter：`url`、`status`（live|redirect|404|stub|no-index）、`h1`、`meta_description`、`word_count`、`has_schema`、`indexed`、`canonical`、`internal_links_in`、`internal_links_out`、`last_crawled`

关键页面：`[[Site Overview]]`、`[[Navigation Structure]]`、`[[Content Gaps]]`、`[[Redirect Map]]`、`[[Keyword Clusters]]`

---

**模式 B：GitHub / 代码仓库**

```
vault/
├── .raw/              # README, git log exports, code dumps
├── wiki/
│   ├── modules/       # one note per module / package / service
│   ├── components/    # reusable components
│   ├── decisions/     # Architecture Decision Records
│   ├── dependencies/  # external deps, versions, risk
│   └── flows/         # data flows, request paths, auth flows
```

modules/ 的 frontmatter：`path`、`status`（active|deprecated|experimental|planned）、`language`、`purpose`、`maintainer`、`depends_on`、`used_by`、`linked_issues`

关键页面：`[[Architecture Overview]]`、`[[Data Flow]]`、`[[Tech Stack]]`、`[[Dependency Graph]]`、`[[Key Decisions]]`

---

**模式 C：商业 / 项目**

```
vault/
├── .raw/              # meeting transcripts, Slack exports, docs
├── wiki/
│   ├── stakeholders/  # people, companies, decision-makers
│   ├── decisions/     # key decisions with rationale and date
│   ├── deliverables/  # milestones, outputs, status
│   ├── intel/         # competitor analysis, market research
│   └── comms/         # synthesized meeting notes
```

decisions/ 的 frontmatter：`status`（active|pending|done|blocked|superseded）、`priority`（1-5）、`date`、`owner`、`due_date`、`context`

关键页面：`[[Project Overview]]`、`[[Stakeholder Map]]`、`[[Decision Log]]`、`[[Competitor Landscape]]`

---

**模式 D：个人 / 第二大脑**

```
vault/
├── .raw/              # journal entries, articles, voice transcripts
├── wiki/
│   ├── goals/         # personal and professional goals
│   ├── learning/      # concepts being mastered
│   ├── people/        # relationships, shared context
│   ├── areas/         # life areas: health, finances, career
│   └── resources/     # books, courses, tools
├── _meta/
│   └── hot-cache.md   # ~500 words of active context
```

goals/ 的 frontmatter：`area`（health|career|finance|creative|relationships|growth）、`priority`、`target_date`、`progress`（0-100）

关键页面：`[[North Star]]`、`[[Weekly Review Template]]`、`[[Annual Goals]]`

---

**模式 E：研究**

```
vault/
├── .raw/              # PDFs, web clips, raw notes
├── wiki/
│   ├── papers/        # paper summaries with key claims
│   ├── concepts/      # extracted concepts, models, frameworks
│   ├── entities/      # people, organizations, datasets
│   ├── thesis/        # evolving synthesis
│   └── gaps/          # open questions, contradictions
```

papers/ 的 frontmatter：`year`、`authors`、`venue`、`key_claim`、`methodology`、`contradicts`、`supports`

关键页面：`[[Research Overview]]`、`[[Key Claims Map]]`、`[[Open Questions]]`、`[[Methodology Comparison]]`

---

**模式 F：书籍 / 课程**

```
vault/
├── .raw/              # chapter notes, highlights, exercises
├── wiki/
│   ├── characters/    # characters, personas, experts
│   ├── themes/        # major themes with evidence
│   ├── concepts/      # domain-specific terms
│   ├── timeline/      # structure, sequence, chapter map
│   └── synthesis/     # your own takeaways and applications
```

concepts/ 的 frontmatter：`source_chapters`、`first_appearance`

关键页面：`[[Book Overview]]`、`[[Theme Map]]`、`[[Character / Expert Index]]`、`[[My Takeaways]]`

### 4.1b —— Vault CLAUDE.md 模板

为一个新项目 vault 做 scaffold 时，在 vault 根目录创建以下文件：

```markdown
# [WIKI NAME] — LLM Wiki

Mode: [MODE A/B/C/D/E/F]
Purpose: [ONE SENTENCE]
Owner: [NAME]
Created: YYYY-MM-DD

## Structure

[PASTE THE FOLDER MAP FROM THE CHOSEN MODE]

## Conventions

- All notes use YAML frontmatter: type, status, created, updated, tags (minimum)
- Wikilinks use [[Note Name]] format — filenames are unique, no paths needed
- .raw/ contains source documents — never modify them
- wiki/index.md is the master catalog — update on every ingest
- wiki/log.md is append-only — new entries go at the TOP, never edit past entries

## Operations

- Ingest: drop source in .raw/, say "ingest [filename]"
- Query: ask any question — Claude reads index first, then drills in
- Lint: say "lint the wiki" to run a health check
```

### 4.2 INGEST —— 单一来源

触发条件：用户把一个文件丢进 `.raw/`，或粘贴一段内容。

1. 完整阅读来源。
2. 与用户讨论关键要点。如果用户说“just ingest it”就跳过。
3. 在 `wiki/sources/` 中创建来源摘要页。
4. 为提到的每个人物/组织/产品/仓库创建或更新 entity 页面。
5. 为重要的想法创建或更新 concept 页面。
6. 更新相关的 domain 页面及其 `_index.md` 子索引。
7. 如果大局发生变化，更新 `wiki/overview.md`。
8. 更新 `wiki/index.md`。为所有新页面添加条目。
9. 用本次 ingest 的上下文更新 `wiki/hot.md`。
10. 追加到 `wiki/log.md`（新条目放在最顶部）：
    ```markdown
    ## [2026-04-07] ingest | Source Title
    - Source: `.raw/articles/filename.md`
    - Summary: [[Source Title]]
    - Pages created: [[Page 1]], [[Page 2]]
    - Pages updated: [[Page 3]], [[Page 4]]
    - Key insight: One sentence on what is new.
    ```
11. 检查矛盾。在两个页面上都用 `> [!contradiction]` callout 标记。

单个来源通常会触及 8-15 个 wiki 页面。

### 4.3 INGEST —— 批量模式

触发条件：用户丢进多个文件，或说“ingest all of these”。

1. 列出所有待处理文件。与用户确认。
2. 按单一 ingest 流程处理每个来源。交叉引用暂缓。
3. 全部来源处理完后：做一遍交叉引用。寻找新来源之间的关联。
4. 在最后一次性更新 index、热缓存和 log，而不是每个来源都更新。
5. 报告：“处理了 N 个来源。创建 X 个页面，更新 Y 个页面。关键关联：……”

批量 ingest 的交互性较弱。对于 30 个以上的来源，每处理 10 个就向用户汇报一次。

### 4.4 QUERY —— 回答问题

1. 先读 `wiki/hot.md`。答案可能就在里面。
2. 读 `wiki/index.md` 找到相关页面。
3. 阅读这些页面（通常 3-5 个，10 个以上就太多了）。
4. 在聊天中综合给出答案。用 wikilink 引用出处。
5. 主动提议把它归档为 `wiki/questions/` 中的一个 wiki 页面。
6. 如果问题暴露了一个空白：“我对 X 掌握得不够。要不要找一个来源？”

### 4.5 LINT —— 健康检查

触发条件：用户说“lint”，或每 10-20 次 ingest 之后。

检查项：孤立页面、失效链接、过时的论断、被提到却缺失的概念页面、缺失的交叉引用、frontmatter 缺口、空章节。

输出：`wiki/meta/lint-report-YYYY-MM-DD.md`。自动修复前先询问。

---

## 5 — 索引与子索引

### wiki/index.md（主索引）

```markdown
---
type: meta
title: "Wiki Index"
updated: 2026-04-07
---
# Wiki Index

## Domains
- [[Domain Name]] — description (N sources)

## Entities
- [[Entity Name]] — role (first: [[Source]])

## Concepts
- [[Concept Name]] — definition (status: developing)

## Sources
- [[Source Title]] — author, date, type

## Questions
- [[Question Title]] — answer summary
```

### 领域子索引

每个领域文件夹都有一个 `_index.md`，只编录该领域的页面目录。

```markdown
---
type: meta
title: "Entities Index"
updated: 2026-04-07
---
# Entities

## People
- [[Person Name]] — role, org

## Organizations
- [[Org Name]] — what they do
```

### wiki/log.md

只追加，不修改。新条目放在最顶部。每个条目：`## [YYYY-MM-DD] operation | title`

解析最近的条目：
```bash
grep "^## \[" wiki/log.md | head -10
```

---

## 6 — 跨项目引用

任何 Claude Code 项目都能读取你的 wiki，而不必复制上下文。

在另一个项目的 CLAUDE.md 中添加：

```markdown
## Wiki Knowledge Base
Path: ~/Documents/Obsidian Vault

When you need context not already in this project:
1. Read wiki/hot.md first (recent context, ~500 words)
2. If not enough, read wiki/index.md (full catalog)
3. If you need domain specifics, read wiki/<domain>/_index.md
4. Only then read individual wiki pages

Do NOT read the wiki for general coding questions, things already in this
project's context, or tasks unrelated to [your domain].
```

这样能把 token 用量压低。热缓存约花 500 token。索引约花 1000 token。单个页面每个约 100-300 token。

---

## 7 — 视觉定制

在 scaffold 期间应用。创建 `.obsidian/snippets/vault-colors.css`：

```css
:root {
  --wiki-1: #4fc1ff;  --wiki-2: #c586c0;  --wiki-3: #dcdcaa;
  --wiki-4: #ce9178;  --wiki-5: #6a9955;  --wiki-6: #d16969;
  --wiki-7: #569cd6;
}

.nav-folder-title[data-path^="wiki/domains"]     { color: var(--wiki-1); }
.nav-folder-title[data-path^="wiki/entities"]    { color: var(--wiki-2); }
.nav-folder-title[data-path^="wiki/concepts"]    { color: var(--wiki-3); }
.nav-folder-title[data-path^="wiki/sources"]     { color: var(--wiki-4); }
.nav-folder-title[data-path^="wiki/questions"]   { color: var(--wiki-5); }
.nav-folder-title[data-path^="wiki/comparisons"] { color: var(--wiki-6); }
.nav-folder-title[data-path^="wiki/meta"]        { color: var(--wiki-7); }
.nav-folder-title[data-path=".raw"]              { color: #808080; opacity: 0.6; }

.callout[data-callout='contradiction'] { --callout-color: 209, 105, 105; --callout-icon: lucide-alert-triangle; }
.callout[data-callout='gap']           { --callout-color: 220, 220, 170; --callout-icon: lucide-help-circle; }
.callout[data-callout='key-insight']   { --callout-color: 79, 193, 255;  --callout-icon: lucide-lightbulb; }
.callout[data-callout='stale']         { --callout-color: 128, 128, 128; --callout-icon: lucide-clock; }
```

启用：Settings > Appearance > CSS Snippets > 刷新 > 打开开关。

### 图谱视图分组（Graph View Groups）

在 Graph View 设置中配置：

| 查询 | 颜色 |
|-------|-------|
| `path:wiki/domains` | 蓝 |
| `path:wiki/entities` | 紫 |
| `path:wiki/concepts` | 黄 |
| `path:wiki/sources` | 橙 |
| `path:wiki/questions` | 绿 |
| `path:.raw` | 灰（变暗） |

---

## 8 — Git 设置

```bash
cd "$VAULT_PATH"
git init
cat > .gitignore << 'EOF'
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.smart-connections/
.obsidian-git-data
.trash/
.DS_Store
node_modules/
EOF
git add -A && git commit -m "Initial vault scaffold"
```

启用 Obsidian Git：Settings > Obsidian Git > Auto backup interval > 15 minutes。

---

## 9 — Dataview 仪表盘

scaffold 之后在 `wiki/meta/dashboard.md` 中创建：

````markdown
---
type: meta
title: "Dashboard"
---
# Wiki Dashboard

## Recent Activity
```dataview
TABLE type, status, updated FROM "wiki" SORT updated DESC LIMIT 15
```

## Seed Pages (Need Development)
```dataview
LIST FROM "wiki" WHERE status = "seed" SORT updated ASC
```

## Entities Missing Sources
```dataview
LIST FROM "wiki/entities" WHERE !sources OR length(sources) = 0
```
````

---

## 10 — 上下文窗口管理

只读取所需的最少内容：

- 先读 `hot.md`。你需要的东西可能已经在里面。
- 其次读 `index.md`。找到相关页面，不要扫描所有内容。
- 做聚焦查找时读领域子索引。
- 每次查询只读 3-5 个页面。10 个以上就太多了。
- 关键词查找用搜索。不要为找一个词而扫整页。
- 做精准编辑用 PATCH。永远不要为改一个字段而重读并重写整个文件。
- 保持 wiki 页面简短。最多 100-300 行。长页面要拆分。
- 除非用户要求，不要把 wiki 内容粘贴进聊天。用 wikilink 引用。

---

## 11 — REST API 速查

运行任何命令前先设置好这些：

```bash
API="https://127.0.0.1:27124"
KEY="your-api-key-here"
```

**读取一个文件：**
```bash
curl -sk -H "Authorization: Bearer $KEY" "$API/vault/wiki/index.md"
```

**创建或替换一个文件：**
```bash
curl -sk -X PUT \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: text/markdown" \
  --data-binary @file.md \
  "$API/vault/wiki/entities/Name.md"
```

**追加到一个文件：**
```bash
curl -sk -X POST \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: text/markdown" \
  --data "- New item" \
  "$API/vault/wiki/log.md"
```

**Patch 一个 frontmatter 字段：**
```bash
curl -sk -X PATCH \
  -H "Authorization: Bearer $KEY" \
  -H "Operation: replace" -H "Target-Type: frontmatter" \
  -H "Target: status" -H "Content-Type: application/json" \
  --data '"mature"' \
  "$API/vault/wiki/concepts/Name.md"
```

**在某个标题下追加：**
```bash
curl -sk -X PATCH \
  -H "Authorization: Bearer $KEY" \
  -H "Operation: append" -H "Target-Type: heading" \
  -H "Target: Connections" -H "Content-Type: text/markdown" \
  --data "- [[New Page]]" \
  "$API/vault/wiki/entities/Name.md"
```

**搜索：**
```bash
curl -sk -X POST \
  -H "Authorization: Bearer $KEY" \
  "$API/search/simple/?query=machine+learning"
```

**Dataview 查询：**
```bash
curl -sk -X POST \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/vnd.olrapi.dataview.dql+txt" \
  --data 'TABLE status FROM "wiki" WHERE status = "seed"' \
  "$API/search/"
```

---

## 12 — Vault CLAUDE.md 模板

为一个新项目（不是这个插件本身）创建 wiki 时，在 vault 根目录创建一个 CLAUDE.md：

```markdown
# [WIKI NAME] — LLM Wiki

Mode: [MODE A/B/C/D/E/F]
Purpose: [ONE SENTENCE]
Owner: [NAME]
Created: YYYY-MM-DD

## Structure

[PASTE THE FOLDER MAP FROM THE CHOSEN MODE]

## Conventions

- All notes use YAML frontmatter: type, status, created, updated, tags (minimum)
- Wikilinks use [[Note Name]] format
- .raw/ contains source documents — never modify them
- wiki/index.md is the master catalog — update on every ingest
- wiki/log.md is append-only — new entries go at the TOP

## Operations

- Ingest: drop source in .raw/, say "ingest [filename]"
- Query: ask any question
- Lint: say "lint the wiki"
```

---

## 13 — 约定

### 命名

- **文件名**：首字母大写、带空格（`Machine Learning.md`）
- **文件夹**：小写、用连字符（`wiki/data-models/`）
- **标签**：小写、分层（`#domain/architecture`）
- **文件名唯一**，这样 wikilink 无需路径即可工作

### 写作风格

- 陈述句、现在时。写 “X uses Y”，不要写 “X basically does Y”。
- 大量加链接。每次提到一个 wiki 页面都加 wikilink。
- 引用来源：`(Source: [[Page]])`。
- 标记不确定性：`> [!gap] This needs more evidence.`
- 标记矛盾：`> [!contradiction] [[Page A]] claims X, but [[Page B]] says Y.`

### 交叉引用

当你更新页面 A 让它提到页面 B 时，检查页面 B 是否也应该反向链接回来。双向链接让图谱视图变得有用。

---

## 14 — Canvas 地图

创建 `.canvas` 文件用于可视化概览：

```json
{
  "nodes": [
    {"id": "1", "type": "file", "file": "wiki/domains/Architecture.md",
     "x": 0, "y": 0, "width": 250, "height": 120, "color": "4"},
    {"id": "2", "type": "file", "file": "wiki/domains/APIs.md",
     "x": 300, "y": 0, "width": 250, "height": 120, "color": "5"}
  ],
  "edges": [
    {"id": "e1", "fromNode": "1", "fromSide": "right",
     "toNode": "2", "toSide": "left", "toEnd": "arrow"}
  ]
}
```

Canvas 节点颜色（Obsidian canvas 颜色码）：1=红，2=橙，3=黄，4=绿，5=青，6=紫。
注意：这些与 wiki 图谱 CSS 配色方案不同。完整的 canvas 颜色表见 `../../skills/canvas/references/canvas-spec.md`。

在 scaffold 期间创建一个领域关系 canvas。随着 wiki 增长而更新。

---

## 小结

你作为 LLM 的工作：
1. 设置 vault（一次性）
2. 根据用户的领域描述 scaffold wiki 结构
3. ingest 来源：阅读、摘要、交叉引用、归档
4. 每次操作后维护热缓存
5. 用“索引 > 相关页面 > 综合”的顺序回答问题
6. 把好的答案归档回 wiki
7. 定期 lint：发现并修复健康问题
8. 永不修改 `.raw/` 来源
9. 始终更新 index、子索引、log 和热缓存
10. 始终使用 frontmatter 和 wikilink

人类的工作：筛选来源、提出好问题、思考其含义。其余一切都由你负责。

---

*基于 Andrej Karpathy 的 LLM Wiki 模式。插件：claude-obsidian，作者 AgriciDaniel / AI Marketing Hub。*
