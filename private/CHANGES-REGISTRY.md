# 私有改动台账

记录本 fork 相对 upstream 的私有定制。**只记 git 看不出的东西** —— 「为什么改」和「升级冲突时怎么处理」。
文件清单用 git 命令即可（把基线版本替换进去）：

```bash
git diff --stat upstream/1.9.2...main
```

每做一处私有定制就在此登记；升级合并冲突时按此表逐条核对，优先保留私有版再并入 upstream 新内容。

| 改动 | 涉及文件/目录 | 为什么 | 升级冲突策略 | 引入版本 |
|------|--------------|--------|--------------|----------|
| 私有维护规范 | `private/README.md` | fork 维护流程，upstream 没有 | 保私有版 | 1.9.2-private.1 |
| 私有目录 | `private/` | 台账/规范 + 译文，与 upstream 隔离 | upstream 无此目录，不冲突 | 1.9.2-private.1 |
| 中文译文 | `private/translations/` | 团队中文参考，不回推 upstream | upstream 无此目录，不冲突 | 1.9.2-private.1 |
| 指针段 | `CLAUDE.md` / `AGENTS.md` | 让 CC/Codex 自动遵循私有规范 | 保私有段，再并入 upstream 新内容 | 1.9.2-private.1 |

## 高冲突文件清单

以下文件最容易和 upstream 撞车，升级合并时重点检查，默认「保私有 + 手动并入 upstream 新内容」：

- `README.md` —— 私有化常改顶部介绍/徽章；upstream 发版也常动。
- `CHANGELOG.md` —— upstream 每次发版必改，几乎必冲突。
- `.claude-plugin/plugin.json` —— 官方版本号字段所在；升级时 `version` 必变。
- `CLAUDE.md` / `AGENTS.md` —— 含私有维护规范指针段，与 upstream 的指令更新会撞。

## 本项目的特殊坑（勘察阶段发现，逐条记录）

- `.obsidian/workspace.json` 被 upstream **故意跟踪**（`.gitignore` 第 2 行注释 "intentionally tracked — ships with pre-configured graph view"）→ **勿删、勿 gitignore**；本机用 `git update-index --skip-worktree .obsidian/workspace.json` 忽略其本地变化。
- `.gitignore` 第 52 行 `WIKI*.md` 通配会**误伤** `private/translations/WIKI.zh.md`（也包括根目录 `WIKI.md` 本身，upstream 已用 `-f` 强制跟踪）→ 新增译文首次 `git add` 需 `git add -f private/translations/WIKI.zh.md`。
- 验证命令为 `make test`（覆盖 address/tiling/boundary/bm25/retrieve/lock/concurrent/mode/contextual 等子项）。
