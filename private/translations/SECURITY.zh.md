---
source: SECURITY.md
source_version: 1.9.2
translated_at: 2026-06-10
---

# 安全策略

## 报告安全问题

如果你在 claude-obsidian 中发现安全问题，请私下报告，而不要公开提交 issue。

**首选方式：** 使用 GitHub 的私密报告功能，访问仓库的 [Security Advisories](../../security/advisories/new) 页面。

**备选方式：** 发邮件至 **agricidaniel@gmail.com**，主题填写 `claude-obsidian security`。

请包含以下信息：
- 对问题的简短描述
- 复现步骤
- 受影响的文件及版本
- 如果你有修复建议，请一并提供

## 响应

你将在 5 个工作日内收到确认回复。修复时间取决于严重程度：

- 严重（数据泄露、命令执行、供应链风险）：7 天内修复
- 高（有条件触发的泄露）：30 天内修复
- 中 / 低：纳入下一次例行发布

## 适用范围

本策略覆盖：
- `skills/`、`agents/`、`scripts/`、`hooks/`、`bin/` 下的插件代码
- `.claude-plugin/` 下的插件清单（manifest）
- 提交前校验器 agent（pre-commit verifier agent）

不在范围内：
- 用户自己撰写的 wiki 页面内容（你的数据由你掌控）
- 插件以 shell 方式调用的第三方工具（Obsidian、defuddle-cli、ollama 等）——请向上游报告
- 需要预先取得用户机器本地访问权限才能触发的问题

## 威胁模型：单租户 vault

claude-obsidian 假定采用**单租户**部署：一个用户、一个 vault、一台机器。若干设计决策都基于这一假设，在多租户或共享 CI 场景下需要做显式加固：

- **`scripts/wiki-lock.sh release`** 会无条件删除锁文件，不管该锁是由哪个进程获取的。这是有意为之——acquire 与 release 通常来自同一台主机上同一个技能的不同 bash 调用，因此绑定 PID 的 release 在正常使用中反而会失败。在共享主机或多用户场景下，任何能写入 `.vault-meta/locks/` 的用户都可能释放掉别人正在进行中的锁。此场景下的缓解措施：将 `.vault-meta/locks/` 的文件系统权限限制为仅 vault 所有者可访问。
- **PostToolUse 自动提交 hook**（`hooks/hooks.json`）以调用 Claude Code 的用户身份运行。它会在每次 Write/Edit 时把 `wiki/`、`.raw/` 和 `.vault-meta/` 路径自动提交到本地仓库。设置 `.vault-meta/auto-commit.disabled`（内容任意）即可按 vault 退出该行为。对于共享仓库，建议完全禁用该 hook，或采用更严格的提交策略。
- **跨进程资源访问**（锁文件、transport 快照、embed 缓存）由文件系统权限管控，而非应用层身份校验。标准的 Linux/macOS 文件权限就是信任边界。

如果你的部署环境中上述任一假设不成立，请在采用前通过上面的安全联系方式与我们取得联系。

## 披露

除非报告者另有意愿，我们会在发布说明（release notes）中致谢报告者。对于遵循本策略、出于善意的报告者，我们不会采取法律行动。
