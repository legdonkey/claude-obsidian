---
source: PRIVACY.md
source_version: 1.9.2
translated_at: 2026-06-10
---

# 隐私

## 数据处理

claude-obsidian 是一个 Claude Code 插件兼 Obsidian vault，完全运行在你
本地的机器上。你的 vault 就是位于你自己文件系统上的纯 Markdown 文件。核心
skill 不会收集、传输或存储任何个人数据，也没有任何遥测、分析或使用情况追踪。

## 哪些内容留在本地

- 摄取来源、回答查询、lint 检查以及更新 hot cache，全部都在你本地的
  Claude Code 会话中运行。
- 所有 wiki 内容（`wiki/`）都是保存在你本地文件系统上的纯 Markdown。
- `/wiki-retrieve` 的 BM25 索引以及可选的基于 ollama 的重排序（reranking）完全
  在本地运行。如果没有显式开启 opt-in 标志，检索数据绝不会离开你的机器。

## 可选的网络出口（opt-in，需经同意授权）

只有当你显式启用时才会发生网络调用。默认情况下一切都在本地进行。

| Feature | Service | Data sent | Gate |
|---------|---------|-----------|------|
| `/wiki-retrieve` contextual prefix | Anthropic API | Wiki 页面分块（用于生成前缀） | 默认关闭。需要在 `scripts/contextual-prefix.py` 上传入 `--allow-egress` 同意标志；否则该层会回退到 PATH 上的 `claude`，或一个完全本地的合成前缀。 |
| `/autoresearch` | Web search + fetch | 你的研究查询以及抓取的 URL | Opt-in；只有当你调用研究循环时才会运行。 |
| `/defuddle` | Web fetch | 你要求它提取的 URL | Opt-in；只有当你调用它时才会运行。 |
| ollama rerank | `localhost` only | 查询 + 候选分块 | 默认本地运行。除非你传入 `--allow-remote-ollama`，否则会拒绝远程 ollama 主机。 |

## 凭据

- API 密钥（例如 `ANTHROPIC_API_KEY`）从环境变量或你本地的 `.env` 中读取，
  绝不硬编码。
- 凭据绝不会被提交到仓库（由 `.gitignore` 拦截）。
- 随附的演示 vault 和配置只附带占位值。

## 安全披露

如需报告安全或隐私问题，请参阅 [`SECURITY.md`](../../SECURITY.md)。
