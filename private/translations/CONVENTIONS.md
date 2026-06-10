# 官方文档中文翻译（私有参考）

upstream 官方文档的中文译文，仅作团队参考，**不回推 upstream**。

## 翻译范围（白名单）

本清单由 privatize-fork 阶段2「翻译范围选择框」扫描本项目文档后确认，是增量翻译的**唯一依据**。
增删翻译范围改这里即可（不要改 skill 流程文件），重跑 skill 会照此刷新。

**翻译（白名单）**：

- `README.md`
- `CONTRIBUTING.md`
- `SECURITY.md`
- `PRIVACY.md`
- `WIKI.md`
- `CLAUDE.md`
- `AGENTS.md`
- `docs/compound-vault-guide.md`
- `docs/dragonscale-guide.md`
- `docs/install-guide.md`
- `docs/methodology-modes-guide.md`

**显式排除（黑名单，不翻）**：

- `CHANGELOG.md`、`docs/releases/*` —— 变更/发版记录，正文是历史流水，翻了价值低且频繁过期。
- `docs/audits/*` —— 内部审计文档。
- `GEMINI.md` —— AI 指令文件，默认排除。（`CLAUDE.md` / `AGENTS.md` 已按团队需要移入白名单作中文参考；译文为 `.zh.md` 放本目录，在子目录、改了名，不会被 AI 当指令加载。）
- `CODE_OF_CONDUCT.md`、`ATTRIBUTION.md` —— 标准开源样板。
- `.github/*`、`.windsurf/*` —— issue/PR 模板与工具配置，非阅读型文档。
- `skills/**/SKILL.md`、`agents/*.md`、`commands/*.md`、`hooks/README.md`、`_templates/*` —— 功能性/操作性文件（给 agent 看的指令、模板），非团队参考文档。
- `wiki/**` —— vault 知识库内容本身（多为英文研究笔记），不是项目文档；如需中文版按需单独处理，默认不翻。
- `.raw/*` —— 源资料，immutable。

## 命名约定

- 译文文件名：`<原文件名去扩展>.zh.md`，例如 `README.zh.md`。
- 译文头部加 frontmatter，标注译自哪个源文件、哪个 upstream 版本：

```yaml
---
source: README.md
source_version: 1.9.2
translated_at: 2026-01-01
---
```

## 防过期

译文会随 upstream 原文更新而过时。升级 upstream 后：

1. 对比每个译文的 `source_version` 与新基线版本。
2. 若源文件在新版本有改动（下面命令输出非空），重译并更新 frontmatter：

```bash
git diff --stat <旧版本>..<新版本> -- <source 路径>
```
