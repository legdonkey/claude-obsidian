# claude-obsidian: Agent Instructions

This repo is a Claude Code plugin **and** an Obsidian vault that builds persistent, compounding knowledge bases using Andrej Karpathy's LLM Wiki pattern. It works with **any AI coding agent** that supports the Agent Skills standard, including Codex CLI, OpenCode, and similar.

Originally built for Claude Code, the skills follow the cross-platform Agent Skills spec. Newer skills (`wiki-fold`, `wiki-ingest`, `wiki-lint`) use only `name` and `description` frontmatter (kepano convention). Some older skills still carry an optional `allowed-tools` field for Claude Code compatibility; cross-platform agents that do not recognize it should ignore it.

## Skills Discovery

All skills live in `skills/<name>/SKILL.md`. Codex / OpenCode / other Agent Skills compatible agents will auto-discover them when you symlink the directory:

```bash
# Codex CLI
ln -s "$(pwd)/skills" ~/.codex/skills/claude-obsidian

# OpenCode
ln -s "$(pwd)/skills" ~/.opencode/skills/claude-obsidian
```

Or run the bundled installer:

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

## Key Conventions

- **Vault root**: the directory containing `wiki/` and `.raw/`
- **Hot cache**: `wiki/hot.md` (read at session start, updated at session end)
- **Source documents**: `.raw/` (immutable: agents never modify these)
- **Generated knowledge**: `wiki/` (agent-owned, links to sources via wikilinks)
- **Manifest**: `.raw/.manifest.json` tracks ingested sources (delta tracking)

## Bootstrap

When the user opens this project for the first time:

1. Read this file (`AGENTS.md`) and the project `CLAUDE.md` for full context
2. Read `skills/wiki/SKILL.md` for the orchestration pattern
3. If `wiki/hot.md` exists, read it silently to restore recent context
4. If the user types `/wiki` (or "set up wiki"), follow the wiki skill's scaffold workflow

## Reference

- Plugin homepage (public canonical): https://github.com/AgriciDaniel/claude-obsidian
- Community early-access mirror (Pro): https://github.com/AI-Marketing-Hub
- Pattern source: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- Cross-reference: https://github.com/kepano/obsidian-skills (authoritative Obsidian-specific skills)

## 私有维护规范（fork 私有化）

本仓库是 `AgriciDaniel/claude-obsidian` 的私有 fork（`origin` = `legdonkey/claude-obsidian`，`upstream` 只读且 push=`DISABLED`）。维护时遵守：

1. **私有内容隔离**：新增私有文件一律进 `private/`（本指针段除外）；不往 upstream 维护的目录里新建私有文件。
2. **必改上游的登记入账**：确需直接改 upstream 文件（配置、版本号、补丁等）时，登记到 [`private/CHANGES-REGISTRY.md`](private/CHANGES-REGISTRY.md) 并定升级冲突策略。
3. **永不 force-push、永不移动已发布 tag**；私有主干只 merge 不 rebase。

完整流程（远端/分支/升级/私有版本号/高冲突清单）见 [`private/README.md`](private/README.md)。中文译文在 `private/translations/`。
