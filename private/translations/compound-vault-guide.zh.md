---
source: docs/compound-vault-guide.md
source_version: 1.9.2
translated_at: 2026-06-10
---

# Compound Vault — v1.7 指南

**状态：** v1.7.0（代号 "Compound Vault refoundation"），发布于 2026-05-17  
**读者对象：** 从 v1.6 升级的用户、新采用者，以及与 v1.7+ 原语集成的 skill 作者  
**配套文档：** [dragonscale-guide.md](../../docs/dragonscale-guide.md)（可选的 Memory 扩展）、[install-guide.md](../../docs/install-guide.md)、[CHANGELOG.md](../../CHANGELOG.md)

---

## 为什么叫 "Compound Vault"

v1.7 系列引入了一个系统名称——**Compound Vault**——用来命名这套架构，与插件名（`claude-obsidian`）区分开来。插件名保留，以延续 SEO 连续性和现有的 4.1k+ stars；系统名则涵盖让这套架构得以运转的 13 个相互协同的 skill。

三句话定位：

> *"Compounding vault, not chat. CLI-native, not chat-window. Methodology-aware, not generic."*

- **复利型 vault（Compounding vault）**——Karpathy 的 LLM Wiki 模式。知识跨会话累积；每次 ingest 都让 wiki 更丰富。
- **CLI 原生（CLI-native）**——Obsidian 1.12 把 `obsidian` 二进制变成了一等公民。v1.7 把它定为默认传输方式，并把 MCP 降级为后备。
- **方法论感知（Methodology-aware）**——在 v1.7 中只是部分实现（modes 将在 v1.8 发布）。但这一定位已经在塑造 v1.7 的范围。

可直接采用的标语候选（用于博客/营销）：

> **"Karpathy's wiki, in your Obsidian."**

---

## 相对 v1.6 有何变化（执行摘要）

v1.7 分四个工作流交付（§3.1 底座 / §3.2 传输 / §3.3 检索 / §3.4 并发），每一个都足够独立，出问题时可以单独回滚。没有破坏性变更——一个升级后什么都不做的 v1.6 vault，行为依然和 v1.6 完全一致。

| 工作流 | 内容 | 原因 | 采用者需做的事 |
|---|---|---|---|
| §3.1 底座 | 3 个 skill 从 soft-defer 升级为 hard-prefer `kepano/obsidian-skills` | 停止与平台所有者竞争 | `claude plugin marketplace add kepano/obsidian-skills`（推荐） |
| §3.2 传输 | 新增 `wiki-cli` skill + `detect-transport.sh` + 决策树 | Obsidian 1.12 CLI 是最快、最安全的写入路径 | 无——首次会话时自动检测 |
| §3.3 检索 | 新增 `wiki-retrieve` skill + contextual prefix + BM25 + cosine rerank | Anthropic 2024 年 9 月研究：检索失败率降低 35-67% | `bash bin/setup-retrieve.sh`（选用） |
| §3.4 并发 | 新增 `wiki-lock.sh` + 4 个 skill 守卫 + hook debounce | 修复潜在的多写者损坏 bug | 无——普遍受益，无需设置 |

---

## §3.1 对 kepano/obsidian-skills 的底座依赖

**它是什么：** 三个 claude-obsidian skill（`obsidian-markdown`、`obsidian-bases`、`canvas`）与 `kepano/obsidian-skills`（由 Obsidian 的 CEO Steph Ango 维护）中的 skill 有重叠。在 v1.6 中我们采取 soft-defer（"如果安装了 kepano，就优先用它"）。在 v1.7 中我们采取 hard-prefer：kepano 是规范来源；我们的副本只是底线。

**原因：** 继续提供平台所有者原语的并行实现，是一场结构性必败之战。kepano marketplace 有 30.5k+ stars；我们有 4.1k+。采用 kepano 作为底座，既表明了对齐姿态，也让我们能把精力投入到无人占据的*工作流*层（ingest、query、lint、autoresearch、save、retrieve）。

**代码库中的变化：**
- `skills/obsidian-markdown/SKILL.md:11`——前言改写为 "This skill is a self-contained fallback. Prefer `kepano/obsidian-skills`."
- `skills/obsidian-bases/SKILL.md:11`——同样的模式。
- `skills/canvas/SKILL.md:14`——同样的模式（json-canvas 规范让位给 kepano；wiki 范围内的工作流仍由 claude-obsidian 负责）。
- `skills/defuddle/SKILL.md:11`——记录为规范来源（kepano 不提供 defuddle skill）。
- `.claude-plugin/marketplace.json`——`recommendedCompanions` 数组列出 `kepano/obsidian-skills`，附带安装提示、理由和仓库链接。

**采用者需做的事：** 运行 `claude plugin marketplace add kepano/obsidian-skills`。即便不装，现有 skill 也能继续工作（本地后备依然可用）。

---

## §3.2 默认传输——带后备链的 Obsidian CLI

**它是什么：** 一个带自动检测的四级传输栈。新 skill `wiki-cli` 记录了 CLI 用法。新脚本 `scripts/detect-transport.sh` 写入 `.vault-meta/transport.json`，供其他 skill 查询。

后备链（从高优先级到低优先级）：
1. **cli**——`obsidian-cli` 二进制（Obsidian 1.12+）。无 MCP 服务器、无 TLS、无插件。
2. **mcp-obsidian**——基于 REST-API 的 MCP 服务器（需要 Local REST API 插件）。自动检测推迟到 v1.7.x。
3. **mcpvault**——基于文件系统的 MCP 服务器（BM25 搜索；无需 Obsidian 插件）。自动检测推迟实现。
4. **filesystem**——直接使用 `Read`/`Write`/`Edit` 工具。始终可用；是底线。

**原因：** v1.6 把四种传输方式平等记录。skill 默认用直接的 `Read`/`Write`。v1.7 收紧了推荐，并把选择变成对 `.vault-meta/transport.json` 的一行查询。

**架构：**

```
detect-transport.sh (run at session start or vault setup)
    │
    └─ writes → .vault-meta/transport.json
                {
                  "preferred": "cli" | "filesystem",
                  "fallback_chain": [...],
                  "available": { cli: {...}, filesystem: {...}, mcp_obsidian: null, mcpvault: null }
                }

skills (wiki-ingest, wiki-query, save, autoresearch, wiki-lint):
    ├─ each has a "## Transport (v1.7+)" section near the top
    ├─ reads transport.json at runtime
    └─ uses obsidian-cli if "preferred": "cli", else Read/Write
```

**采用者需做的事：** 无——检测会自动运行，并在 7 天后刷新。强制刷新：`bash scripts/detect-transport.sh --force`。若要手动固定到某个 MCP 传输，编辑 `.vault-meta/transport.json` 并设置 `"manual_override": true`，这样检测脚本就不会动你的修改。

**参见：** [`wiki/references/transport-fallback.md`](../../wiki/references/transport-fallback.md) 了解完整决策树，以及 [`skills/wiki-cli/SKILL.md`](../../skills/wiki-cli/SKILL.md) 了解逐操作的用法配方。

---

## §3.3 混合检索流水线——wiki-retrieve（选用）

**它是什么：** 一个新的选用型 skill，用 chunk 级检索取代 v1.6 的静态 `Read(hot.md) → Read(index.md) → Read(N pages)` 查询路径，方法是在带上下文前缀的 chunk 上做 BM25 + cosine rerank。它把 Anthropic 的 [Sept 2024 Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval) 模式实现为 agent-skill 的管道。

**原因：** 只要答案藏在某个具体段落里，page 级粒度就会输给 chunk 级粒度。Anthropic 测得：上下文前缀让检索失败率降低 35%，混合 BM25+vector 降低 49%，再加一个 reranker 降低 67%。v1.7 实现了 contextual + sparse + rerank 这套栈。（单独的稠密向量阶段在 v1.7.x 路线图上。）

**架构：**

```
INGEST (one-time + incremental):

  wiki/<page>.md
       │
       ▼
  scripts/contextual-prefix.py
       │  ├─ chunks on paragraph boundaries (~500 tokens target, 200 char overlap)
       │  └─ generates 1-2 sentence prefix per chunk
       │       tier 1: ANTHROPIC_API_KEY  → Anthropic API (Haiku, prompt-cached
       │                                    when body ≥ ~16 KB / Haiku 4.5 floor)
       │       tier 2: claude on PATH    → `claude -p` subprocess
       │       tier 3: synthetic          → frontmatter title + first paragraph
       │
       ▼  .vault-meta/chunks/<address>/chunk-NNN.json

  scripts/bm25-index.py build
       └─ inverted index over chunks' contextualized_text → .vault-meta/bm25/index.json

QUERY:

  query string
       │
       ▼
  scripts/retrieve.py "<query>" --top 5
       ├─ scripts/bm25-index.py query → top-20 candidates by BM25
       ├─ scripts/rerank.py            → cosine on nomic-embed-text via ollama
       │     (no-op if ollama unreachable; BM25 order preserved)
       └─ page-address dedupe          → final top-5 with absolute_path
       │
       ▼
  caller (wiki-query / autoresearch) reads cited pages and synthesizes
```

**特性门控：** 其他 skill 通过以下方式检测 wiki-retrieve：

```bash
[ -x scripts/retrieve.py ] && [ -d .vault-meta/chunks ] && [ -f .vault-meta/bm25/index.json ]
```

如果检测失败，skill 会回退到 v1.6 的旧读取顺序。该 skill 绝不会破坏基础插件。

**成本上限：** 按 Anthropic 公布的数字，约每 1,000 篇文档 ~$12（Haiku + prompt caching，tier 1）。Tier 2（claude CLI）在金钱上免费但更慢。Tier 3（synthetic）免费且自洽封闭；会丢掉大部分上下文收益，但 BM25 + rerank 仍然有效。

**采用者需做的事：**

```bash
bash bin/setup-retrieve.sh         # full provisioning (auto-picks prefix tier)
bash bin/setup-retrieve.sh --no-llm  # force tier 3 (zero LLM dependency)
bash bin/setup-retrieve.sh --check   # diagnostics only; no provisioning
```

设置完成后，`wiki-query` 的 standard/deep 模式会自动使用新流水线。Quick 模式（仅 hot.md）保持不变。

**参见：** [`skills/wiki-retrieve/SKILL.md`](../../skills/wiki-retrieve/SKILL.md) 了解完整的 skill 规格、配方参考，以及 v1.7.x 路线图（BGE cross-encoder、Cohere Rerank、单独的稠密向量阶段）。

---

## §3.4 多写者安全——wiki-lock（核心）

**它是什么：** 通过 `scripts/wiki-lock.sh` 实现的逐文件咨询式锁。每次 wiki 页面写入之前都必须先 `wiki-lock acquire <path>`，之后再 `wiki-lock release <path>`。

**原因：** v1.6 有一个潜在的损坏 bug。`skills/wiki-ingest/SKILL.md:259-264` 把 "single-writer only" 记录为一种约定，但实际的页面写入路径没有任何强制。两个并行的子 agent 写同一个 wiki 页面时，可能会悄无声息地互相覆盖。Karpathy-LLM-Wiki-Stack 的 README 明确警告过这一点。v1.7 堵上了这个漏洞。

**设计（基于年龄，而非 flock 风格）：**

`flock(2)` 咨询锁会在持锁进程退出时释放。这不符合我们的模型——`acquire` 和 `release` 是同一个 skill 发出的两次独立 bash 调用（每次 Bash 工具调用都是各自独立的短命进程——任何一方的 PID 都活不到有意义的时长）。因此 `wiki-lock.sh` 采用：

- **原子的 noclobber 写锁文件**（在 POSIX 文件系统上无竞态风险）。
- **基于 epoch 的年龄陈旧判定**：超过 `STALE_AFTER_SEC`（默认 60）的锁会被自动回收。崩溃的持锁者会在 ≤60s 内自动解锁，无需人工干预。
- **允许跨进程释放**：`release` 就是 `rm -f`（不要求 PID 匹配）。我们信任 skill 作者会释放自己获取的锁。`wiki-lock clear-stale --max-age 0` 命令是规范的恢复路径。
- **锁文件里的 PID 仅供参考**（对 `list` 和调试有帮助）。

**skill 集成：**

四个 skill 新增了 "## Concurrency (v1.7+)" 章节，配方如下：

```bash
if bash scripts/wiki-lock.sh acquire wiki/concepts/Foo.md; then
  # … do the write via the §Transport-selected method …
  bash scripts/wiki-lock.sh release wiki/concepts/Foo.md
else
  # rc=75 = EX_TEMPFAIL = another writer in flight. Retry once after 2s;
  # if still held, log to wiki/log.md and skip this page.
  sleep 2
  bash scripts/wiki-lock.sh acquire wiki/concepts/Foo.md && {
    # write …
    bash scripts/wiki-lock.sh release wiki/concepts/Foo.md
  } || echo "skipped wiki/concepts/Foo.md (locked)"
fi
```

**hook 集成：** `hooks/hooks.json` 的 PostToolUse 现在会在有锁被持有时延后 `git add`。可防止多 agent ingest 期间产生撕裂的提交。当 `wiki-lock.sh` 不存在时会优雅地直接通过。

**采用者需做的事：** 无——`wiki-lock.sh` 在 v1.7 中是核心（无需选用）。不遵守 acquire/release 模式的子 agent 会与任何其他写者竞争（一如既往——但现在有工具可以修复它）。

**测试覆盖：** `tests/test_wiki_lock.sh`（14 个自洽断言）和 `tests/test_concurrent_write.sh`（关键的正确性闸门——10 个并行 worker，无丢失，无串行错乱）。`make test-concurrent` 和 `make test-lock`。

**参见：** [`scripts/wiki-lock.sh`](../../scripts/wiki-lock.sh) 头部注释了解完整语义，以及 `skills/wiki-ingest/SKILL.md` §Concurrency 了解规范的集成模式。

---

## Skill 清单（v1.7）

总计 13 个 skill。v1.7 新增：`wiki-cli`、`wiki-retrieve`。

| Skill | 状态 | 角色 |
|---|---|---|
| `wiki` | core | 设置 / 脚手架 / 子 skill 路由 |
| `wiki-ingest` | core | 源 → 带交叉引用的 wiki 页面 |
| `wiki-query` | core | 问答（若已安装则使用 wiki-retrieve） |
| `wiki-lint` | core | 健康检查（孤儿、死链、地址、tiling） |
| `wiki-fold` | DragonScale Mech 1 | 抽取式日志汇总 |
| `wiki-cli` | **v1.7 新增（§3.2）** | Obsidian CLI 传输封装 |
| `wiki-retrieve` | **v1.7 新增（§3.3，选用）** | Contextual + BM25 + rerank |
| `save` | core | 把对话归档为 wiki 笔记 |
| `autoresearch` | core | 迭代式网络研究 → wiki |
| `canvas` | core（让位给 kepano json-canvas） | 可视化 wiki 层 |
| `defuddle` | core（规范来源） | 网页清理器 |
| `obsidian-markdown` | core（让位给 kepano） | Obsidian Flavored Markdown 参考 |
| `obsidian-bases` | core（让位给 kepano） | Bases YAML 参考 |

---

## 脚本清单（v1.7）

| 脚本 | 状态 | 角色 |
|---|---|---|
| `allocate-address.sh` | DragonScale Mech 2 | 原子 c-NNNNNN 分配器（flock） |
| `tiling-check.py` | DragonScale Mech 3 | 基于 embedding 的重复 lint（fcntl） |
| `boundary-score.py` | DragonScale Mech 4 | autoresearch 的前沿评分 |
| `detect-transport.sh` | **v1.7 新增（§3.2）** | 传输检测 → transport.json |
| `contextual-prefix.py` | **v1.7 新增（§3.3）** | Chunk + 3 层前缀生成 |
| `bm25-index.py` | **v1.7 新增（§3.3）** | 稀疏倒排索引（flock） |
| `rerank.py` | **v1.7 新增（§3.3）** | 通过 ollama 做 cosine rerank（缓存上用 fcntl） |
| `retrieve.py` | **v1.7 新增（§3.3）** | 混合检索编排器 |
| `wiki-lock.sh` | **v1.7 新增（§3.4）** | 逐文件咨询锁（noclobber） |

---

## 测试（v1.7）

`make test` 运行 7 个测试套件。全部自洽——零网络、零 ollama、零 LLM 调用。

| 目标 | 文件 | 断言数 | 覆盖范围 |
|---|---|---|---|
| `make test-address` | `tests/test_allocate_address.sh` | ~10 | DragonScale Mech 2 |
| `make test-tiling` | `tests/test_tiling_check.py` | ~15 | DragonScale Mech 3 |
| `make test-boundary` | `tests/test_boundary_score.py` | ~35 | DragonScale Mech 4 |
| `make test-bm25` | `tests/test_bm25_index.py` | ~30 | tokenize、BM25 单调性、IDF |
| `make test-retrieve` | `tests/test_retrieve.py` | 22 | cosine、rerank、端到端子进程 |
| `make test-lock` | `tests/test_wiki_lock.sh` | 14 | acquire、release、基于年龄的回收 |
| `make test-concurrent` | `tests/test_concurrent_write.sh` | 6 | **关键的多写者正确性闸门** |

"自洽测试不变式（hermetic test invariant）"得到保留：`make test` 中没有任何东西需要网络、ollama 或任何 API key。可选流水线（用 Anthropic API 的 contextual prefix、用 ollama cosine 的 rerank）通过 mock 和优雅回退来测试。

---

## v1.7 不是什么

- 不是重写。DragonScale Mechanisms 1-4 得到保留且未改动。
- 不是破坏性变更。不运行 setup-retrieve.sh 的 v1.6 vault 行为不变（除了 wiki-lock 集成，它普遍受益且不增加任何设置）。
- 不是付费插件。License 仍为 MIT。
- 不是 GUI Obsidian 插件外壳。推迟到 v2.5+（Claudian/deivid11 封装模式是 2026 年 5 月差距分析 backlog 中的第 7 项）。
- 不是多 vault 联邦。推迟到 v2.x。

---

## 路线图指引

2026 年 5 月的差距分析识别出 20 个 backlog 项。v1.7 交付第 1、2、3、4 项（按价值/工作量计算属于最高四分位）外加潜在 bug 修复。下一批里程碑（取决于用户优先级排序）：

- **v1.8**——方法论模式（通过 `wiki-mode` 提供 LYT / PARA / Zettelkasten / Generic）+ 周期性回顾（`wiki-review`）。关闭差距 #6 + #11。
- **v1.9**——多模态 ingest 适配器（通过 `wiki-ingest-multimodal` 支持 YouTube、PDF、EPUB、图片 OCR）。关闭差距 #8 + #12。
- **v2.0**——NotebookLM 级别的衍生输出（通过 `wiki-derive` 提供音频、测验、闪卡、学习指南）。关闭差距 #5 + #9 + #14。

完整计划：`~/.claude/plans/read-in-full-the-hidden-sun.md`。

---

## 另见

- [CHANGELOG.md](../../CHANGELOG.md)——v1.7.0 条目
- [docs/dragonscale-guide.md](../../docs/dragonscale-guide.md)——DragonScale Memory 扩展（Mechanisms 1-4）
- [docs/install-guide.md](../../docs/install-guide.md)——安装
- [wiki/references/transport-fallback.md](../../wiki/references/transport-fallback.md)——传输决策树
- [wiki/concepts/DragonScale Memory.md](../../wiki/concepts/DragonScale%20Memory.md)——规格
- Anthropic Contextual Retrieval: https://www.anthropic.com/news/contextual-retrieval
- kepano/obsidian-skills: https://github.com/kepano/obsidian-skills
- Karpathy LLM Wiki gist: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
