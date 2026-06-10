---
source: CLAUDE.md
source_version: 1.9.2
translated_at: 2026-06-10
---

# claude-obsidian — Claude + Obsidian Wiki Vault

本文件夹既是一个 Claude Code 插件，也是一个 Obsidian vault。

**插件名：** `claude-obsidian`（v1.7+ "Compound Vault" —— 见 [docs/compound-vault-guide.md](../../docs/compound-vault-guide.md)；v1.8+ 新增方法论模式 —— 见 [docs/methodology-modes-guide.md](../../docs/methodology-modes-guide.md)）
**Skills：** `/wiki`、`/wiki-ingest`、`/wiki-query`、`/wiki-lint`、`/wiki-cli`（v1.7）、`/wiki-retrieve`（v1.7，可选启用）、`/wiki-mode`（v1.8）
**Vault 路径：** 本目录（可直接在 Obsidian 中打开）

## 这个 Vault 用来做什么

本 vault 演示了 LLM Wiki 模式 —— 一个为 Claude + Obsidian 打造的、可持续累积的持久化知识库。投入任意来源、提出任意问题，wiki 都会随每一次会话变得更丰富。

## Vault 结构

```
.raw/           source documents — immutable, Claude reads but never modifies
wiki/           Claude-generated knowledge base
_templates/     Obsidian Templater templates
_attachments/   images and PDFs referenced by wiki pages
```

## 如何使用

把一个源文件放进 `.raw/`，然后告诉 Claude："ingest [filename]"。

提出任意问题。Claude 会先读索引，再深入到相关页面。

运行 `/wiki` 来搭建一个新 vault 或检查安装状态。

每 10-15 次 ingest 后运行一次 "lint the wiki"，以发现孤立页面和缺口。

## 跨项目访问

要从另一个 Claude Code 项目引用本 wiki，请在那个项目的 CLAUDE.md 中加入：

```markdown
## Wiki Knowledge Base
Path: /path/to/this/vault

When you need context not already in this project:
1. Read wiki/hot.md first (recent context, ~500 words)
2. If not enough, read wiki/index.md
3. If you need domain specifics, read wiki/<domain>/_index.md
4. Only then read individual wiki pages

Do NOT read the wiki for general coding questions or things already in this project.
```

## 插件 Skills

| Skill | 触发方式 |
|-------|---------|
| `/wiki` | 安装、搭建、路由到子 skill |
| `ingest [source]` | 单个或批量源 ingest |
| `query: [question]` | 基于 wiki 内容作答 |
| `lint the wiki` | 健康检查 |
| `/save` | 把当前对话归档为一篇结构化的 wiki 笔记 |
| `/autoresearch [topic]` | 自主研究循环：搜索、抓取、综合、归档 |
| `/canvas` | 视觉层：向 Obsidian canvas 添加图片、PDF、笔记 |
| `/wiki-cli`（v1.7） | Obsidian CLI 传输封装；桌面端默认的写入路径 |
| `/wiki-retrieve`（v1.7） | 混合检索：contextual + BM25 + cosine-rerank（通过 `bash bin/setup-retrieve.sh` 可选启用） |
| `/wiki-mode`（v1.8） | 方法论模式（LYT / PARA / Zettelkasten / Generic）。通过 `bash bin/setup-mode.sh` 设置；由 wiki-ingest / save / autoresearch 消费，用于为新页面路由 |
| `/think`（v1.9） | 把 10 原则思考循环（OBSERVE-OBSERVE-LISTEN-THINK-CONNECT-CONNECT-FEEL-ACCEPT-CREATE-GROW）作为一个可调用的工作流。适用于架构决策、审计、复盘、含糊的用户请求。其他每个 skill 都有一个 "How to think" 附录，把这套框架映射到它各自的工作上 |

## Transport（v1.7+）

`scripts/detect-transport.sh` 在首次运行时写入 `.vault-meta/transport.json`，并每周刷新。各 skill 在修改 vault 之前会先查询它。回退链：Obsidian CLI → mcp-obsidian → mcpvault → filesystem（始终可用的兜底）。决策树：[wiki/references/transport-fallback.md](../../wiki/references/transport-fallback.md)。

## 并发（v1.7+）

`scripts/wiki-lock.sh` 提供按文件的咨询锁（advisory lock），以保证多写入者 ingest 的安全。每一次 wiki 页面写入都应由 `wiki-lock acquire`/`release` 守护。stale-after 默认为 60s；按设计允许跨进程释放。在持有锁期间，PostToolUse hook 会推迟 `git add`。这堵上了 v1.6 中潜在的多写入者损坏漏洞。

## 方法论模式（v1.8+）

通过 `bash bin/setup-mode.sh` 为 vault 选择一种组织风格。共有四种模式：**generic**（v1.7 默认 —— 不持立场）、**LYT**（Linking Your Thinking —— MOCs + 原子笔记）、**PARA**（Projects/Areas/Resources/Archives）、**Zettelkasten**（带时间戳的 ID、扁平结构、密集链接）。该模式写入 `.vault-meta/mode.json`（默认被 gitignore；用 `git add -f` 提交）。`wiki-ingest`、`save` 和 `autoresearch` 在归档新页面前会先查询 `python3 scripts/wiki-mode.py route <type> "<name>"` —— 消费方 skill 无需做任何特殊处理。完整指南：[docs/methodology-modes-guide.md](../../docs/methodology-modes-guide.md)。这堵上了 2026 年 5 月 compass 工件中的优先级缺口 5。

## 提交前校验器（v1.7.1+）

为一个非琐碎的工作流暂存（stage）改动之后、但在运行 `git commit` 之前，派发 `verifier` agent（`agents/verifier.md`）。它会读取 `git diff --cached`，应用 /best-practices 六刀法（six-cut）+ agent 内核，并以四个层级（BLOCKER / HIGH / MEDIUM / LOW）返回发现，附带 file:line 引用。该 agent 只拥有只读工具（Read、Grep、Glob、Bash）—— 它能检查但永远无法修改，因此它的输出纯属建议性质。这堵上了 v1.7 审计所揭示的闭环漏洞：代码从 worker 直接走到 commit，中间没有独立的校验器环节，这正是 BLOCKER B1（数据外泄同意缺口，data-egress consent gap）得以溜过去的原因。复盘见 `docs/audits/v1.7.0-audit-2026-05-17.md` §10。

## MCP（可选）

如果你配置了 MCP server，Claude 就能直接读写 vault 笔记。
安装说明见 `skills/wiki/references/mcp-setup.md`。

## 发布博客文章

在切出一个新版本（git tag + `gh release create`）之后，运行：

```
/release-blog
```

它会在 https://agricidaniel.com/blog/ 上生成一篇博客文章，处理封面图生成、SEO 元数据、FAQ schema、内部链接、sitemap/llms.txt 更新、Vercel 部署，以及 Google 索引。

## 私有维护规范（fork 私有化）

本仓库是 `AgriciDaniel/claude-obsidian` 的私有 fork（`origin` = `legdonkey/claude-obsidian`，`upstream` 只读且 push=`DISABLED`）。维护时遵守：

1. **私有内容隔离**：新增私有文件一律进 `private/`（本指针段除外）；不往 upstream 维护的目录里新建私有文件。
2. **必改上游的登记入账**：确需直接改 upstream 文件（配置、版本号、补丁等）时，登记到 [`private/CHANGES-REGISTRY.md`](private/CHANGES-REGISTRY.md) 并定升级冲突策略。
3. **永不 force-push、永不移动已发布 tag**；私有主干只 merge 不 rebase。

完整流程（远端/分支/升级/私有版本号/高冲突清单）见 [`private/README.md`](private/README.md)。中文译文在 `private/translations/`。
