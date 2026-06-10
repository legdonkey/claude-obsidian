---
source: docs/dragonscale-guide.md
source_version: 1.9.2
translated_at: 2026-06-10
---

# DragonScale Memory 指南

DragonScale Memory 是 `claude-obsidian` 的一个可选扩展。它增加了一些保守的辅助能力：日志汇总（rollup）、稳定的页面地址、重复页面检查（lint），以及前沿主题建议。请先从 [docs/install-guide.md](../../docs/install-guide.md) 开始。要了解设计规格与设计理由，请阅读 [wiki/concepts/DragonScale Memory.md](../../docs/../wiki/concepts/DragonScale%20Memory.md)。

本页内容紧贴 `v1.6.0` 中已发布的实际行为。它解释了安装会创建什么、每个机制实际做什么、它需要什么，以及如何在不卸载仓库的前提下安全地关闭它。

## DragonScale 是什么

### 范围与可选启用状态

DragonScale 是 wiki 的内存层（memory-layer）扩展。它涵盖汇总、确定性页面 ID、重复检测，以及一条面向 `/autoresearch` 的可选主题选择路径。它对于基础 vault 而言不是必需的。

如果你从不运行 `bash bin/setup-dragonscale.sh`，基础安装与原有的技能行为将保持不变。仓库使用特性检测（feature detection），让 DragonScale 可以保持可选状态，而不会变成硬依赖。

概念页（concept page）的范围比本指南更广。本指南是面向操作的。当设计规格与实现细节存在差异时，在日常行为上请以已发布的脚本和技能为准。

### 1.6.0 中发布了什么

版本 `1.6.0` 将全部四个 DragonScale 机制作为可选特性发布：

- 机制 1，Fold Operator：`skills/wiki-fold/`
- 机制 2，确定性页面地址：`scripts/allocate-address.sh`，外加 `wiki-ingest` 与 `wiki-lint` 集成
- 机制 3，语义平铺检查（Semantic Tiling Lint）：`scripts/tiling-check.py`，外加 `wiki-lint` 集成
- 机制 4，边界优先的 Autoresearch：`scripts/boundary-score.py`，外加 `skills/autoresearch/SKILL.md` 的主题选择逻辑

发布脉络请查阅 `CHANGELOG.md`，快速上手视角请看 [docs/install-guide.md](../../docs/install-guide.md)，完整设计背景请看 [wiki/concepts/DragonScale Memory.md](../../docs/../wiki/concepts/DragonScale%20Memory.md)。

## 启用之前

### 基础安装要求

DragonScale 是一个附加组件，不是基础安装的替代品。请先按照 [docs/install-guide.md](../../docs/install-guide.md) 完成正常的 vault 安装。

至少要做到：

- 克隆仓库或安装插件
- 运行 `bash bin/setup-vault.sh`
- 将该文件夹作为 Obsidian vault 打开
- 使用 `/wiki` 来搭建脚手架或继续安装

DragonScale 安装脚本接受一个可选参数，即 vault 路径：

```bash
bash bin/setup-dragonscale.sh
```

```bash
bash bin/setup-dragonscale.sh /path/to/vault
```

如果你省略该路径，脚本会使用从 `bin/` 推断出的仓库根目录。

### 通用前置条件：flock

`flock` 是通用前置条件。机制 2 在 `scripts/allocate-address.sh` 中直接使用它来保护 `.vault-meta/.address.lock`。机制 3 从 Python 中使用 flock，在缓存 I/O 周围保护 `.vault-meta/.tiling.lock`。

快速检查：

```bash
command -v flock
```

如果上面没有任何输出，请在依赖 DragonScale 之前先安装 `flock`。在 Linux 上它通常已经存在。在 macOS 上它通常来自 `util-linux`。

如果缺少 `flock`，安装脚本仍然能创建文件，但地址分配器和平铺缓存路径将不可靠。请把这种情况当作阻断项（blocker）处理。

### 机制 3 的额外前置条件：python3、ollama、nomic-embed-text

机制 3 是唯一带有完整本地嵌入（embeddings）栈的机制。你需要 `python3`、`ollama`，以及已拉取进 ollama 的 `nomic-embed-text` 模型。

有用的检查：

```bash
command -v python3
```

```bash
curl -sS http://127.0.0.1:11434/api/version
```

```bash
ollama pull nomic-embed-text
```

安装脚本不会安装上述任何一项。它只检查并报告状态。机制 4 需要 `python3`，但不需要 ollama。机制 1 和机制 2 两者都不需要。

### 当可选依赖缺失时会发生什么

DragonScale 的设计目标是失败时安全关闭（fail closed）或干净地空操作（no-op）。

如果缺少 `python3`：

- 机制 3 无法运行
- 机制 4 无法运行
- 机制 1 和机制 2 仍然可用

如果 ollama 不可达，`scripts/tiling-check.py` 退出码为 `10`。如果 ollama 可达但未安装 `nomic-embed-text`，退出码为 `11`。`wiki-lint` 应当将这些情况视为语义平铺的跳过条件，而不是中断其余 lint 流程的理由。

如果边界辅助脚本（boundary helper）失败，`/autoresearch` 会回退到正常的“询问用户”主题路径。它不会强行给出候选列表，也不会即兴编造一个主题。

如果从未运行过 DragonScale 安装，`wiki-ingest` 和 `wiki-lint` 会保持它们非 DragonScale 的行为。

## 安装

### 运行 bin/setup-dragonscale.sh

运行：

```bash
bash bin/setup-dragonscale.sh
```

该脚本是幂等的。重复运行是安全的，它不会覆盖已经创建过的运行时文件。

在配置状态之前，它会验证：

- `scripts/allocate-address.sh`
- `scripts/tiling-check.py`
- `skills/wiki-fold/SKILL.md`

如果其中任何一项缺失，安装会停止并提示你重新安装插件。

安装会做什么：

- 使 `scripts/allocate-address.sh` 可执行
- 使 `scripts/tiling-check.py` 可执行
- 如有需要则创建 `.vault-meta/`
- 如果缺失则创建地址、平铺，以及遗留基线（legacy-baseline）状态文件
- 如果缺失则创建 `.raw/.manifest.json`
- 在最后运行健全性检查（sanity checks）

安装不会做什么：

- 安装 ollama
- 拉取 `nomic-embed-text`
- 把地址回填（backfill）到旧页面上
- 运行一次 fold
- 运行语义平铺
- 重写你现有的 wiki 页面

### 它创建哪些文件和状态

安装会配置少量运行时状态。

在 `.vault-meta/` 中，它会创建：

- `address-counter.txt`
- `tiling-thresholds.json`
- `legacy-pages.txt`

在 `.raw/` 中，它会创建：

- `.manifest.json`

`address-counter.txt` 从 `1` 开始，因此在全新 vault 中，下一个被保留的页面地址将是 `c-000001`。

`tiling-thresholds.json` 以 `error: 0.90`、`review: 0.80` 和 `calibrated: false` 为种子值。这些是保守的初始区间，并非针对你的 vault 校准过的真实值。

`legacy-pages.txt` 会得到一个上线标记注释（rollout marker comment）：

```text
# rollout: YYYY-MM-DD
```

`wiki-lint` 使用该基线来区分遗留页面与上线之后的页面，以便进行地址强制检查。

`.raw/.manifest.json` 初始时带有空的 `sources` 和 `address_map` 对象。ingest 技能维护该文件。`.raw/` 下的源文档保持不可变。

### 如何验证安装

安装脚本本身已经执行了健全性检查，但你自己再验证几项也是有用的。

在不保留地址的情况下查看下一个地址：

```bash
./scripts/allocate-address.sh --peek
```

检查运行时状态是否存在：

```bash
ls -1 .vault-meta
```

在不计算嵌入的情况下检查平铺就绪状态：

```bash
python3 ./scripts/tiling-check.py --peek
```

检查边界辅助脚本：

```bash
python3 ./scripts/boundary-score.py --top 5
```

如果你的 vault 很小或集成度很高，边界辅助脚本可能报告没有正分前沿页面（positive-score frontier pages）。这仍然是一次有效的运行。

## 机制 1：Fold Operator

### 它做什么

fold operator 是一种日志汇总。它读取 `wiki/log.md` 中最近的 `2^k` 条条目，并在 `wiki/folds/` 下生成一个抽取式（extractive）的 fold 页面。

fold 是追加式的。它不会删除、移动或重写子条目。fold 是抽取式的。输出中的每一项结论和主题都必须能追溯到某条子日志条目。

当前发布的技能是有意做得很窄的。它支持对原始日志条目进行扁平 fold。分层的 fold-of-folds 行为仍在当前技能的范围之外，尽管概念规格讨论了堆叠 fold。

对于给定的范围，fold ID 是确定性的：

```text
fold-k{K}-from-{EARLIEST-DATE}-to-{LATEST-DATE}-n{COUNT}
```

这带来了结构上的幂等性。如果完全相同的 fold 已经存在，该技能应当停止，而不是写入一个重复项。

### 何时使用它

当日志已经积累了一批连贯的工作，而你想要一个比一长串原始条目更易扫读的检查点页面时，就使用 fold。

典型场景：

- 围绕某一主题的若干次 ingest 之后
- 一阵研究会话集中爆发之后
- 在扁平的 `wiki/log.md` 变得越来越难用之前

不要把 fold 当作垃圾回收。它们做的是汇总，不会通过删除来压缩。

示例命令：

```text
fold the log, dry-run k=3
```

这是请求对 `2^3 = 8` 条条目做一次 dry-run。

### Dry-run 模式与 commit 模式

Dry-run 是默认模式，并且只输出到 stdout。这一点很重要，因为该仓库为写操作配置了 PostToolUse 钩子。

在 dry-run 模式下：

- 不写入任何文件
- 不触发任何自动提交钩子
- 你会在终端输出中得到提议的 fold 内容

在 commit 模式下：

- fold 页面被写入 `wiki/folds/`
- `wiki/index.md` 被更新
- `wiki/log.md` 得到一条新的 fold 条目

技能文档预期 commit 模式下有三次独立的写操作，所以来自钩子的三次自动提交是正常的。

示例 commit 命令：

```text
fold the log, commit k=3
```

请先运行一次 dry-run。只有在 fold 内容看起来正确时才提交。

要在不卸载 DragonScale 的情况下禁用机制 1，停止调用 `wiki-fold` 即可。已有的 fold 页面可以保留在 vault 中，如果你不再需要它们，也可以手动删除。

## 机制 2：确定性页面地址

### 地址格式与上线策略

机制 2 在 frontmatter 中分配稳定地址。已发布的格式是：

```yaml
address: c-000042
```

`c-` 表示创建顺序计数器（creation-order counter）。数字部分以零填充到六位。这不是内容哈希。规格中明确指出，已发布的地址是确定性且稳定的，但不是内容可寻址的（content-addressable）。

上线基线是 `2026-04-23`。在采用 DragonScale 之后，上线之后的非元（non-meta）页面应当带有地址。遗留页面在你执行一次有意的回填之前是豁免的。

该辅助脚本有三种真实模式：

```bash
./scripts/allocate-address.sh
```

```bash
./scripts/allocate-address.sh --peek
```

```bash
./scripts/allocate-address.sh --rebuild
```

默认模式保留并打印下一个地址。`--peek` 是只读的。`--rebuild` 会根据观察到的最高 `c-NNNNNN` 重新计算计数器。

示例命令：

```bash
./scripts/allocate-address.sh --peek
```

### ingest 和 lint 如何使用它

`wiki-ingest` 仅在 `./scripts/allocate-address.sh` 可执行且 `./.vault-meta` 存在时启用地址分配。如果两个条件都满足，新的非元页面会在 frontmatter 中获得 `address:`。如果不满足，ingest 会在不分配地址的情况下继续。

`wiki-lint` 仅在 `./scripts/allocate-address.sh` 可执行且 `./.vault-meta/address-counter.txt` 存在时启用地址校验。如果这些条件满足，lint 会检查地址格式、唯一性、计数器与 `--peek` 的一致性、上线之后页面上缺失的地址，以及 `.raw/.manifest.json` 中 `address_map` 的一致性。

单写者（single-writer）规则在这里很重要。分配器使用 `flock`，但 ingest 技能仍然规定阶段 2（Phase 2）只能单写者。不要从多个会话或子代理（sub-agents）并行运行会分配地址的 ingest。

技能文档中有一条硬性规则值得重申。绝不要直接编辑 `.vault-meta/address-counter.txt`。只能通过 `scripts/allocate-address.sh` 来改动它。

要在不卸载的情况下禁用机制 2：

1. 停止运行依赖地址分配的 ingest
2. 如果你想让特性检测关闭，移除 `.vault-meta/`
3. 停止使用 `./scripts/allocate-address.sh`

已有的 `address:` 字段可以保留在页面上。如果该特性被禁用，它们会成为惰性元数据。

## 机制 3：语义平铺检查

### 它检查什么

机制 3 是一个基于嵌入的重复页面检测器。它扫描 `wiki/` 下的 markdown 文件，并排除：

- `wiki/folds/`
- `wiki/meta/`
- 常见的元文件名，例如 `index.md`、`log.md`、`hot.md`、`overview.md`、`dashboard.md`、`Wiki Map.md` 和 `getting-started.md`
- 带有 `type: meta` 的文件
- 带有 `type: fold` 的文件
- 符号链接，或逃逸出 vault 根目录的路径

它为每个被纳入的页面计算一个嵌入，按余弦相似度（cosine similarity）逐对比较，并按区间（bands）给出候选的重叠项。

默认区间：

- `>= 0.90` 为 error
- `0.80 - 0.90` 为 review
- `< 0.80` 为 pass

该辅助脚本绝不会自动合并页面。它只报告供审查的候选项。

示例命令：

```bash
python3 ./scripts/tiling-check.py --peek
```

这会给出结构化诊断，而不计算嵌入。

### 本地嵌入要求

默认情况下，该辅助脚本只信任位于 `http://127.0.0.1:11434` 的本地 ollama 端点。远程 ollama 端点需要一个显式的覆盖标志（override flag），因为页面正文会被作为嵌入输入发送出去。

远程覆盖示例：

```bash
python3 ./scripts/tiling-check.py --allow-remote-ollama --peek
```

正常的就绪路径是本地的：

1. 已安装 `python3`
2. ollama 在 localhost 可达
3. `nomic-embed-text` 已安装进 ollama

重要的退出码：

- `0` 成功
- `10` ollama 不可达
- `11` 模型缺失

`wiki-lint` 的编写会将这些视为跳过条件。

### 校准与空操作行为

已发布的阈值是保守的种子值，不是校准过的真实值。技能文档要求对每个 vault 做一次性的手动校准。在你完成校准之前，预期会同时出现假阴性（false negatives）和假阳性（false positives）。

该辅助脚本还具有有意的空操作行为。如果缺少 ollama 或模型，它会以跳过码退出。它不会伪造结果。

有用的命令：

```bash
python3 ./scripts/tiling-check.py --peek
```

```bash
python3 ./scripts/tiling-check.py --rebuild-cache
```

```bash
python3 ./scripts/tiling-check.py --report wiki/meta/tiling-report-YYYY-MM-DD.md
```

`--report` 是真实存在的，并且被限制在 vault 内的路径。当你想要一份保存下来的报告时使用它。当你只想要就绪状态和诊断时使用 `--peek`。

要在不卸载的情况下禁用机制 3：

1. 停止运行 `python3 ./scripts/tiling-check.py`
2. 停止在 `wiki-lint` 中使用语义平铺路径
3. 如果你不需要 ollama 或模型，就不要配置它们

注意 `.vault-meta/` 是机制 2、3、4 共享的门控（gate）。不要为了单独禁用机制 3 而移除它，否则你也会关闭地址分配和边界优先的 autoresearch。平铺缓存位于 `.vault-meta/` 下，但在不调用辅助脚本时是惰性的。

## 机制 4：边界优先的 Autoresearch

### 它做什么

机制 4 对 wiki 图（graph）中的前沿页面进行评分。已发布的公式是：

```text
boundary_score(p) = (out_degree(p) - in_degree(p)) * recency_weight(p)
```

在实践中，高分页面向外指向许多可评分的页面，接收相对较少的入链，并且更新得足够近，仍然像是前沿（frontier）。

该辅助脚本读取 `wiki/**/*.md`，构建 wikilink 图，并将排序后的结果输出到 stdout 或 JSON。它有意只输出到 stdout。与平铺辅助脚本不同，它没有 `--report PATH` 模式。

示例命令：

```bash
python3 ./scripts/boundary-score.py --json --top 5
```

这正是 autoresearch 技能用于生成候选项的确切命令。

### 议程控制（agenda-control）的注意事项

这条注意事项在规格和技能文档中都有明确说明。

这是议程控制，而非纯粹的内存。

机制 4 不仅仅是描述 vault。它会影响代理（agent）接下来可能去研究什么。这跨越了内存与规划的边界。

项目让它保持可选，并诚实地标注它。如果你只想要严格的内存层子集，就省略这条路径。不要在没有主题的情况下使用 `/autoresearch`，或者干脆不要设置并调用边界评分器。

### `/autoresearch` 在有它和没它时的行为

当机制 4 可用，并且仅当 `/autoresearch` 在没有主题的情况下被调用时，该技能会：

1. 检查 `scripts/boundary-score.py`
2. 检查 `./.vault-meta`
3. 检查 `python3`
4. 运行 `./scripts/boundary-score.py --json --top 5`
5. 将排名靠前的前沿页面作为候选主题呈现
6. 让用户选择、用自由文本覆盖，或拒绝

如果该辅助脚本以非零退出、返回无效 JSON，或返回空的 `results` 数组，该技能会回退。

在没有机制 4，或在回退之后，`/autoresearch` 只会询问：

```text
What topic should I research?
```

辅助脚本负责建议。用户仍然负责决定。

要在不卸载的情况下禁用机制 4：

1. 停止运行 `python3 ./scripts/boundary-score.py`
2. 使用带显式主题的 `/autoresearch [topic]`
3. 如果你不想要前沿建议，就避免无主题的 `/autoresearch` 路径

注意 `.vault-meta/` 是机制 2、3、4 共享的门控。不要为了单独禁用机制 4 而移除它。评分器本身是只读的，并且不使用任何共享状态；禁用它只意味着不去调用它。

## 操作策略

### 单写者规则

DragonScale 假定地址分配路径只有单个写者。分配器受 flock 保护，这能防止计数器出现简单的竞态。它并不会把整个 wiki 变成一个安全的多写者系统。

ingest 技能在这里说得很明确。不要从多个 Claude 会话或子代理并行运行会分配地址的 ingest。

安全的运行策略是：

- 同一时间只有一个活跃的 ingest 写者
- 同一时间只有一条地址分配器路径
- 不直接手动编辑计数器状态

机制 1 由人工调用，容易串行化。机制 3 在缓存 I/O 上使用锁。机制 4 是只读的。

### 特性检测与优雅回退

DragonScale 的设计是被特性检测的，而不是被假定存在的。

`wiki-ingest` 仅在分配器可执行且 `.vault-meta/` 存在时才分配地址。
`wiki-lint` 仅在分配器存在且 `.vault-meta/address-counter.txt` 存在时才校验地址。
`wiki-lint` 仅在辅助脚本存在且 `python3` 可用时才运行语义平铺，然后从 `--peek` 解释就绪状态。
`autoresearch` 仅在辅助脚本存在、`.vault-meta/` 存在且 `python3` 存在时才使用边界优先选择。

当这些条件不满足时，仓库会回退到更早的行为。这就是预期的操作姿态。

## 故障排查

### 缺少 flock

如果缺少 `flock`，请先修复它。症状可能包括不安全的地址分配路径，或一个无法正确加锁的平铺缓存路径。

检查：

```bash
command -v flock
```

如果它不存在，请为你的系统安装提供它的软件包，然后重新运行：

```bash
bash bin/setup-dragonscale.sh
```

不要通过直接编辑 `.vault-meta/address-counter.txt` 来绕过这个问题。

### 缺少 ollama 或模型

这只会阻塞机制 3，不会阻塞 DragonScale 的其余部分。

检查 ollama 可达性：

```bash
curl -sS http://127.0.0.1:11434/api/version
```

检查平铺就绪状态：

```bash
python3 ./scripts/tiling-check.py --peek
```

如果该辅助脚本退出码为 `10`，说明 ollama 不可达。如果退出码为 `11`，请拉取模型：

```bash
ollama pull nomic-embed-text
```

然后重新运行：

```bash
python3 ./scripts/tiling-check.py --peek
```

记住机制 4 不需要 ollama。如果你只想要边界优先的 autoresearch，`python3` 就足够了。

### 安全回滚 / 禁用路径

你不需要卸载仓库就能关闭 DragonScale。使用最小的、符合你需求的回滚方式：

- 机制 1：停止调用 `wiki-fold`。它不使用共享状态。
- 机制 2：停止使用 `./scripts/allocate-address.sh`。已有的 `address:` frontmatter 字段会作为普通内容保留。
- 机制 3：停止运行 `python3 ./scripts/tiling-check.py`，并停止在 `wiki-lint` 中调用语义平铺路径。`.vault-meta/` 下的缓存在不使用时是惰性的。
- 机制 4：停止运行 `python3 ./scripts/boundary-score.py`，并避免无主题的 `/autoresearch` 路径。评分器是只读的；禁用就是不去调用它。

`.vault-meta/` 是机制 2、3、4 共享的门控。移除它会同时禁用这三个，而不是只禁用一个。

如果你想一次性关闭基于安装的各个机制的 DragonScale 特性检测，请移除 `.vault-meta/`：

```bash
rm -rf .vault-meta
```

然后停止调用 DragonScale 专属的辅助脚本和技能。这会让你的正常 wiki 内容保持完好。它不会删除 fold 页面，也不会从 frontmatter 中剥离已有的 `address:` 字段。除非你选择手动清理，否则它们会作为普通内容保留。

如果你之后想让 DragonScale 回来，请重新运行：

```bash
bash bin/setup-dragonscale.sh
```
