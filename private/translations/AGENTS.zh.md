---
source: AGENTS.md
source_version: 1.9.2
translated_at: 2026-06-10
---

# claude-obsidian：Agent 指令

本仓库既是一个 Claude Code 插件，**又**是一个 Obsidian vault，它采用 Andrej Karpathy 的 LLM Wiki 模式来构建持久、可累积复利的知识库。它适用于**任何**支持 Agent Skills 标准的 AI 编码 agent，包括 Codex CLI、OpenCode 及类似工具。

这些 skill 最初是为 Claude Code 构建的，但都遵循跨平台的 Agent Skills 规范。较新的 skill（`wiki-fold`、`wiki-ingest`、`wiki-lint`）只使用 `name` 和 `description` 这两个 frontmatter 字段（kepano 约定）。部分较旧的 skill 仍带有一个可选的 `allowed-tools` 字段，用于兼容 Claude Code；不识别该字段的跨平台 agent 应将其忽略。

## Skills Discovery

所有 skill 都位于 `skills/<name>/SKILL.md`。当你软链接该目录后，Codex / OpenCode / 其他兼容 Agent Skills 的 agent 会自动发现它们：

```bash
# Codex CLI
ln -s "$(pwd)/skills" ~/.codex/skills/claude-obsidian

# OpenCode
ln -s "$(pwd)/skills" ~/.opencode/skills/claude-obsidian
```

或者运行随附的安装脚本：

```bash
bash bin/setup-multi-agent.sh
```

## Available Skills

| Skill | Trigger phrases |
|---|---|
| `wiki` | `/wiki`, set up wiki, scaffold vault |
| `wiki-ingest` | ingest, ingest this url, ingest this image, batch ingest |
| `wiki-query` | query, what do you know about, query quick:, query deep: |
| `wiki-lint` | lint the wiki, health check, find orphans |
| `wiki-fold` | fold the log, run a fold, log rollup (DragonScale Mechanism 1, opt-in) |
| `save` | /save, file this conversation |
| `autoresearch` | autoresearch, autonomous research loop |
| `canvas` | /canvas, add to canvas, create canvas |
| `defuddle` | clean this url, defuddle |
| `obsidian-markdown` | obsidian syntax, wikilink, callout |
| `obsidian-bases` | obsidian bases, .base file, dynamic table |

## 关键约定

- **Vault 根目录**：包含 `wiki/` 和 `.raw/` 的目录
- **热缓存（Hot cache）**：`wiki/hot.md`（会话开始时读取，会话结束时更新）
- **源文档（Source documents）**：`.raw/`（不可变：agent 永不修改这些文件）
- **生成的知识（Generated knowledge）**：`wiki/`（由 agent 拥有，通过 wikilink 链接到源文件）
- **清单（Manifest）**：`.raw/.manifest.json` 跟踪已 ingest 的源（增量跟踪）

## Bootstrap

当用户首次打开本项目时：

1. 阅读本文件（`AGENTS.md`）和项目的 `CLAUDE.md` 以获取完整上下文
2. 阅读 `skills/wiki/SKILL.md` 以了解编排模式
3. 如果 `wiki/hot.md` 存在，静默读取它以恢复近期上下文
4. 如果用户输入 `/wiki`（或 “set up wiki”），按照 wiki skill 的 scaffold 工作流执行

## Reference

- 插件主页（公开权威源）：https://github.com/AgriciDaniel/claude-obsidian
- 社区抢先体验镜像（Pro）：https://github.com/AI-Marketing-Hub
- 模式来源：https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- 交叉引用：https://github.com/kepano/obsidian-skills（权威的 Obsidian 专用 skill）

## 私有维护规范（fork 私有化）

本仓库是 `AgriciDaniel/claude-obsidian` 的私有 fork（`origin` = `legdonkey/claude-obsidian`，`upstream` 只读且 push=`DISABLED`）。维护时遵守：

1. **私有内容隔离**：新增私有文件一律进 `private/`（本指针段除外）；不往 upstream 维护的目录里新建私有文件。
2. **必改上游的登记入账**：确需直接改 upstream 文件（配置、版本号、补丁等）时，登记到 [`private/CHANGES-REGISTRY.md`](../../private/CHANGES-REGISTRY.md) 并定升级冲突策略。
3. **永不 force-push、永不移动已发布 tag**；私有主干只 merge 不 rebase。

完整流程（远端/分支/升级/私有版本号/高冲突清单）见 [`private/README.md`](../../private/README.md)。中文译文在 `private/translations/`。
