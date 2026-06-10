---
source: CONTRIBUTING.md
source_version: 1.9.2
translated_at: 2026-06-10
---

# 为 claude-obsidian 做贡献

感谢你有兴趣改进这个插件。claude-obsidian 是一个小而专注的项目；符合其理念的贡献会很快被合入。

## 理念

每一次改动都受三条约束塑造：

1. **先读后写。** 每次贡献都从阅读受影响的文件、它们的测试以及它们的调用方开始。删除和新增一样，常常会破坏既有假设。
2. **能用的最小单元。** 不做投机性的抽象。复杂度是挣来的，不是预先假定的。一个抽象落地前，至少要有三个真实调用方。
3. **失败即规格。** 每一个新的失败模式都需要显式处理。不可信输入、网络调用和状态变更都需要一个明确的“影响范围（blast radius）”答案。

完整的内核位于 [`/best-practices`](https://github.com/AgriciDaniel/best-practices)（一个可组合的 Claude Code skill）。位于 [`agents/verifier.md`](../../agents/verifier.md) 的预提交校验 agent 会对非琐碎改动强制执行它。

## 工作流

### 1. 先开 issue（针对非琐碎改动）

对于错别字修复、文档澄清或单行改动，可以直接提 PR。对于更大的改动：

- 使用 bug-report 或 feature-request 模板开一个 issue
- 描述问题、提议的改动以及影响范围（blast radius）
- 等到拿到肯定（thumbs-up）后再投入时间

### 2. Fork + 分支

贡献在公开的规范仓库（canonical repo）上接受。在 GitHub 上 fork [`AgriciDaniel/claude-obsidian`](https://github.com/AgriciDaniel/claude-obsidian)，然后：

```bash
git clone https://github.com/<your-username>/claude-obsidian.git
cd claude-obsidian
git checkout -b your-feature-name
```

> ℹ️ 公开仓库（`AgriciDaniel/claude-obsidian`）是唯一的规范信源（source of truth）。所有 PR 都应针对它提出。从抢先体验镜像（`AI-Marketing-Hub/claude-obsidian`）工作的 AI Marketing Hub Pro 成员也应将 PR 提向公开规范仓库，这样所有贡献都汇聚到一处。

分支命名：`fix/...`、`feat/...`、`docs/...`、`refactor/...`。

### 3. 在本地搭建环境

该插件可以在任何能运行 Claude Code 的地方运行。用于开发时：

```bash
# 运行封闭（hermetic）测试套件 —— 提交 PR 前必须执行
make test

# 可选：配置 v1.7 检索流水线（需同意 API 出网 egress）
bash bin/setup-retrieve.sh

# 可选：选择一种方法论模式，用于测试 v1.8 的路由
bash bin/setup-mode.sh
```

### 4. 进行改动

遵循“六刀”内核（six-cut kernel）：
- 阅读你要改动的每一个文件
- 给新标识符起名时，假设下一个读者怀有敌意
- 能用的最小单元
- 删的要多于加的（当进行替代时）
- 用证据而非直觉（为新行为编写测试）
- 文档化失败模式 + 撤销（undo）方案

如果你新增了 skill、agent、脚本或 hook，也要在 `tests/` 下增加一个测试。这 9 个封闭测试套件是这个项目的安全网。

### 5. 运行测试

```bash
make test
```

全部 9 个套件都必须通过（约 1240 条断言）。测试是封闭的：不联网、不用 ollama、不调 Anthropic API。如果你的改动新增了网络调用，请将其置于一个 `--consent`/`--allow-egress`/环境变量模式之后，对齐 `scripts/contextual-prefix.py` 或 `scripts/tiling-check.py` 的做法。

### 6. 运行校验器（可选但推荐）

对于多文件改动：

```bash
# 在 git add 之后、git commit 之前，派发 verifier agent
# （Claude Code：对暂存的 diff 调用 agents/verifier.md）
```

校验器会对你暂存的 diff 应用“六刀 + agent 内核”，并以 BLOCKER / HIGH / MEDIUM / LOW 几个层级返回发现项。提交前请先处理 BLOCKER + HIGH。

### 7. 提交

提交信息约定（Conventional Commits）：

```
<type>(<scope>): <short description>

<longer body if needed>
```

类型：`feat`、`fix`、`docs`、`chore`、`refactor`、`test`、`perf`、`style`。

示例：
```
fix(wiki-mode): close path-traversal in route_path safe_name

Sanitize routing input via safe_name() before string-concat into
the vault-relative path. Adds 9 hermetic test assertions for traversal
vectors. Closes audit finding S1.
```

### 8. 更新 CHANGELOG.md

在 `## [Unreleased]` 下增加一条记录（如该小节不存在则创建）。遵循 [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) 格式：

```markdown
## [Unreleased]

### Added
- Description of new feature

### Fixed
- Description of bug fix
```

维护者会在发布时把你的记录移到对应版本的小节中。

### 9. 提交 PR

使用 PR 模板。必填字段：
- 改动了什么
- 为什么
- 如何测试的（如相关，请粘贴 `make test` 的输出）
- 如适用，附上审计/校验结果

PR 会对照内核进行评审。请预期会收到关于命名、范围和失败模式覆盖度的反馈。

## 测试标准

- **封闭（Hermetic）。** 不联网、不依赖外部服务、不依赖用户特定的环境。
- **快速。** 全套件在一分钟内跑完。
- **确定性。** 相同输入 → 相同输出，每次都一样。
- **有文档。** 每个测试断言的是一个有名字的行为，而非巧合。

如果你无法让测试做到封闭，请在 issue 中询问 mock（模拟）替代方案是否可接受。

## 我们不会合并什么

- 没有为新行为编写测试的代码
- 投机性的抽象（尚无真实调用方）
- 没有显式同意（consent）门控的网络调用
- 未经清洗就用用户输入构造路径（参见 `scripts/wiki-mode.py:safe_name`）
- 在未文档化失败模式的情况下 shell out（调起外部命令）的 skill/agent
- 在没有书面理由的情况下破坏现有测试套件的任何改动

## 安全

发现了漏洞？请勿公开开 issue。披露方式见 [SECURITY.md](../../SECURITY.md)。

## 许可证

通过贡献，你同意你的贡献依据 [MIT License](../../LICENSE) 进行授权。

## 行为准则

参与受 [Contributor Covenant Code of Conduct](../../CODE_OF_CONDUCT.md) 约束。
