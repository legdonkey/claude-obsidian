---
source: docs/methodology-modes-guide.md
source_version: 1.9.2
translated_at: 2026-06-10
---

# 方法论模式指南 —— v1.8.0

**状态：** v1.8.0 GA（2026-05-17）
**范围：** 为你的 vault 选定一种组织风格，并据此路由新页面。
**起源：** 关闭 2026 年 5 月 compass 文档中的优先级缺口 5。

---

## TL;DR

挑一个符合**你**思维方式的模式：

| 你的思维方式是…… | 选择 |
|---|---|
| 主题集群 + 通过跟随链接来导航 | **LYT** |
| 进行中的项目 vs 持续的职责 vs 参考资料 | **PARA** |
| 带唯一 ID 的原子化论断 + 密集链接 | **Zettelkasten** |
| 不需要方法论 / 想要 v1.7 默认行为 | **Generic** |

```bash
bash bin/setup-mode.sh           # interactive
bash bin/setup-mode.sh --mode lyt   # non-interactive
```

选定之后，`wiki-ingest`、`save` 和 `autoresearch` 会在决定把新页面归档到哪里之前先查阅该模式。已有文件**不会**被移动；模式只影响今后的归档。

---

## 为什么会有方法论模式

2026 年 5 月的 compass 文档识别出 5 个优先级缺口。claude-obsidian v1.7 关闭了其中 4 个（基底对齐、默认传输、混合检索、多写入者安全），并把第 5 个——方法论支持——推迟到了 v1.8。

审计 §9 的轴线评估在 2026 年 5 月把方法论支持判为**平局（TIE）**：在 Claude+Obsidian 领域里，没有别家把它作为一等公民技能（first-class skill）来交付。Ideaverse Pro 2.0（售价 200 美元的付费 vault）以一种带主张的结构交付了 LYT，但那是一个 vault，而非一套技能集。PARA、Zettelkasten 以及模式感知路由则完全无人覆盖。

v1.8.0 关闭了这一缺口。本次发布之后，按 compass 框架衡量，claude-obsidian 在 **7 条轴线中的 5 条上排名 #1**（compounding wiki、多写入者安全、检索架构、许可证开放性、方法论支持）。剩余的 2 条（GUI 人机工学、衍生输出）需要更大体量的发布（GUI 需 v2.5+，derive 需 v2.0）。

---

## 四种模式

### Generic（默认）

**理念：** 不强加任何方法论。与 v1.6/v1.7 相同。

**归档约定：**
- `wiki/sources/<slug>.md` —— 已摄取的源文档
- `wiki/entities/<Name>.md` —— 人物、组织、产品（保留大小写）
- `wiki/concepts/<Name>.md` —— 概念与框架
- `wiki/sessions/<date>-<topic>.md` —— 来自 `/save` 的会话笔记

**何时使用：**
- 你正从 v1.7 迁移过来，且希望行为零变化
- 你还不想绑定某种方法论
- 你有自己的组织直觉，希望工具尽量少给主张

**优点：** 零学习曲线；契合 v1.7 的肌肉记忆；灵活。
**缺点：** 没有可依靠的主张；在大型 vault 中容易蔓延失控。

---

### LYT（Linking Your Thinking —— Nick Milo）

**理念：** 组织的基本单元是 **MOC**（Map of Content，内容地图）。原子化笔记平铺在同一个文件夹下；MOC 把笔记链接成集群。你通过跟随链接来导航，而不是浏览文件夹。

**归档约定：**
- `wiki/mocs/<topic>-moc.md` —— 某个主题集群的内容地图
- `wiki/notes/<atomic-note>.md` —— 所有原子化笔记平铺（无子文件夹）
- 每条原子化笔记的 frontmatter `mocs:` 字段中至少有一个 MOC
- 新摄取的内容落到 `wiki/notes/`；消费方技能同时更新相关的 MOC

**模板**（位于 `skills/wiki-mode/templates/lyt/`）：
- `moc-template.md` —— 带 core-notes / adjacent-MOCs / open-questions 区块的 MOC 脚手架
- `atomic-template.md` —— 带 MOC 反向链接的原子化笔记

**何时使用：**
- 中到大型知识库（>100 条笔记）
- 你以概念集群和知识图谱来思考
- 你是 LYT 的实践者，或想成为其中一员

**优点：** 扩展性极佳；导航随增长而愈发丰富；知识结构显式化。
**缺点：** 需要时刻更新 MOC 的自律；若缺乏好用的搜索，平铺的 notes 文件夹会显得混乱。

---

### PARA（Tiago Forte）

**理念：** 按**可行动性（actionability）**而非主题来组织。进行中的工作放进 Projects（有截止日期 + 成果），持续的职责放进 Areas（无截止日期），参考资料放进 Resources（按主题），已完成/不活跃的工作放进 Archives。

**归档约定：**
- `wiki/projects/<project-name>/<note>.md` —— 进行中的项目
- `wiki/projects/inbox/<note>.md` —— 新摄取内容 + 会话笔记落到这里待分拣
- `wiki/areas/<area-name>/<note>.md` —— 持续的职责
- `wiki/resources/<topic>/<note>.md` —— 参考资料
- `wiki/resources/incoming/<note>.md` —— 新源落到这里待按主题分类
- `wiki/resources/people/<Name>.md` —— 实体页面
- `wiki/resources/concepts/<Name>.md` —— 概念页面
- `wiki/archives/<year>/<note>.md` —— 已完成的项目、已落幕的 areas

**模板**（位于 `skills/wiki-mode/templates/para/`）：
- `project-template.md` —— 带 status / deadline / outcome / next-action 的项目
- `area-template.md` —— 带 scope / standards / review cadence 的 area
- `resource-template.md` —— 带 topic + sources 的参考资料

**何时使用：**
- 工作流密集的用户
- 管理众多项目的知识工作者
- 偏 GTD 风格的实践者
- 任何读过 Tiago Forte《Building a Second Brain》的人

**优点：** 项目生命周期显式化；活跃内容与参考内容清晰分离；契合知识工作者的真实运作方式。
**缺点：** 需要周期性复查，把已完成的项目移到 archives；"incoming" 桶需要被处理。

---

### Zettelkasten（Niklas Luhmann 的卡片盒）

**理念：** 原子化笔记、唯一 ID、密集的双向链接。没有文件夹。每条笔记只回答一个想法。笔记之间通过 ID 引用彼此找到对方。

**归档约定：**
- `wiki/<YYYYMMDDHHMMSSffffff>-<slug>.md` —— 平铺在 wiki/ 下，使用带时间戳的 ID（20 位数字 = 日期 + 微秒，抗碰撞）
- 每条笔记在 frontmatter 中带有 `id:`、`parent_id:`（可选）、`child_ids:`（可选）
- 无子目录；wiki/ 根目录即整个 vault
- 所有组织关系都通过笔记正文中的 `parent_id` / `child_ids` / `[[ID]]` 引用来表达

**模板**（位于 `skills/wiki-mode/templates/zettel/`）：
- `atomic-template.md` —— 带父/子 ID + 推理过程 + 来源的原子化论断

**何时使用：**
- 学者与研究者
- 构建永久性知识产物的长期思考者
- 任何读过 Sönke Ahrens《How to Take Smart Notes》的人
- 偏好高自律、小归档面的人

**优点：** 链接密度最高；鼓励原子化思考；历经数十年仍能很好地沉淀。
**缺点：** 自律曲线最陡；若缺乏好用的搜索，平铺的文件列表令人望而生畏；基于 ID 的引用不如基于名称的助记性强。

---

## 模式如何与其他技能交互

集成是**自动的**——一旦你设定了模式，`wiki-ingest`、`save` 和 `autoresearch` 会在每一个新页面上查阅它。你永远不必为此操心。

| 技能 | 它做什么 | 模式如何影响它 |
|---|---|---|
| `wiki-ingest` | 归档新的源/实体/概念页面 | router 按模式决定目标文件夹 |
| `save` | 把当前对话归档为会话笔记 | router 决定落点：`wiki/sessions/`（generic）、`wiki/notes/` + MOC 更新（LYT）、`wiki/projects/inbox/`（PARA），或 `wiki/<ID>-session-...`（Zettel） |
| `autoresearch` | 在研究循环后归档综述页面 | router 决定落点：`wiki/concepts/`（generic）、`wiki/notes/` + 主题 MOC（LYT）、`wiki/resources/<topic>/`（PARA），或 `wiki/<ID>-...`（Zettel） |

router（`scripts/wiki-mode.py route <type> "<name>"`）是唯一的事实来源。技能自己不计算路径；它们调用 router 并使用其返回值。

---

## 之后切换模式

切换模式是**安全的，但不会自动迁移**：

1. 运行 `bash bin/setup-mode.sh`（或非交互地用 `--mode <new-mode>`）
2. 新模式被写入 `.vault-meta/mode.json`
3. 已有文件保留在原始位置并继续可用
4. 新文件按新模式归档
5. （可选的手动步骤）用你的文件管理器或 `git mv` 把已有文件迁移到新结构

**为何不自动迁移：** wiki 装的是你的思考。自动改写路径可能破坏 wikilinks、丢失数据，或让你措手不及。手动迁移迫使你就"什么契合新方法论 vs 什么留在原位"做出明确决定。

**专门针对 LYT 迁移：** 切换到 LYT 之后，运行 `lint the wiki`（技能：wiki-lint）来识别那些适合纳入 MOC 的孤儿页面。

---

## 模式配置文件

`.vault-meta/mode.json` 是当前生效的模式声明。它**默认被 gitignore**——该文件被当作主机特定的运行时配置。若要把你的模式选择跨机器/协作者提交：

```bash
git add -f .vault-meta/mode.json
git commit -m "chore: declare vault mode as <mode>"
```

文件 schema：

```json
{
  "schema_version": 1,
  "mode": "lyt|para|zettelkasten|generic",
  "configured_at": "2026-05-17T00:00:00Z",
  "config": {
    "lyt": {"moc_folder": "wiki/mocs/", "notes_folder": "wiki/notes/"},
    "para": {"projects_folder": "...", "areas_folder": "...", "resources_folder": "...", "archives_folder": "..."},
    "zettelkasten": {"id_format": "YYYYMMDDHHMMSSffffff", "no_folders": true, "root_folder": "wiki/"},
    "generic": {"sources_folder": "wiki/sources/", "entities_folder": "wiki/entities/", "concepts_folder": "wiki/concepts/", "sessions_folder": "wiki/sessions/"}
  }
}
```

`config` 块始终包含全部 4 种模式。生效的模式由 `mode` 指明。如果你想使用非默认约定，可以在你的 `mode.json` 中覆盖各模式的文件夹路径。

---

## 何时**不要**使用模式感知

- **极小的 vault**（<20 条笔记）：此时组织的开销尚不值得。坚持用 generic。
- **你并未主动想去组织的 vault**：如果你不在乎方法论，就别选一个。Generic 才是诚实的选择。
- **跨项目共享的 vault**（依据全局 CLAUDE.md 的 `/save` 约定）：位于 `~/Documents/Obsidian Vault/` 的个人 vault 有它自己的组织选择；项目的模式 router 只作用于项目自身的 `wiki/`。

---

## 从这里出发的路线图

v1.8.0 关闭了优先级缺口 5。compass 文档的全貌：

| 轴线（依据审计 §9） | v1.7.2 状态 | v1.8.0 状态 | 通向 LEAD 之路 |
|---|---|---|---|
| Compounding wiki primitive | #1 | #1 | ✓ |
| Multi-writer safety | #1 | #1 | ✓ |
| Retrieval architecture（免费层） | #1 | #1 | ✓ |
| License / openness | #1 | #1 | ✓ |
| **方法论支持** | TIE | **#1** ← v1.8.0 关闭 | ✓ |
| Derivative outputs（音频/视频/测验） | NO | NO | v2.0（wiki-derive） |
| GUI / 安装人机工学 | NO | NO | v2.5+（Community Plugin fork） |

v1.8.0 之后：按 compass 框架衡量，**7 条轴线中的 5 条排名 #1**。剩余 2 条轴线需要跨多次发布的努力：
- **v1.9** —— 多模态摄取（YouTube / PDF / EPUB / 图像 OCR）
- **v2.0** —— `wiki-derive` 技能：音频概览、测验生成、学习指南、思维导图综合（对标 NotebookLM）
- **v2.5+** —— Community Plugin GUI 外壳（触达主流 Obsidian 用户）

---

## 交叉引用

- [`skills/wiki-mode/SKILL.md`](../../skills/wiki-mode/SKILL.md) —— 技能本身
- [`scripts/wiki-mode.py`](../../scripts/wiki-mode.py) —— router + 配置助手
- [`bin/setup-mode.sh`](../../bin/setup-mode.sh) —— 交互式设置
- [`tests/test_wiki_mode.py`](../../tests/test_wiki_mode.py) —— 封闭测试套件（15 条断言）
- [`docs/compound-vault-guide.md`](../../docs/compound-vault-guide.md) —— v1.8 所基于的 v1.7 综合指南
- v1.7.0 审计 §9 轴线 6：[`docs/audits/v1.7.0-audit-2026-05-17.md`](../../docs/audits/v1.7.0-audit-2026-05-17.md)
