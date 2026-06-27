# 论文写作 Skill 体系设计概要（修订版 v2）

> 仿照 Reasonix code skill 的完整决策链结构，搭建一套论文写作 skill 体系。
> 覆盖 research → write → review → revise → submit 全流程闭环。
> 基底能力来自 academic-research-skills 和 nature-skills，但全部经过 Reasonix 适配层封装。

---

## 一、课题目标

搭建一套完整的论文写作 skill 体系，与现有 code skill 决策链对等，在论文写作领域实现同等水平的流程约束和决策自动化。"不只是论文写作 skill 集合，而是论文工作流操作系统。"

核心原则：两个基底仓库是**能力原材料**，不是架构模板。它们必须被 Reasonix 决策链"驯化"，而不是让 Reasonix 迁就它们原来的 Claude/Codex 工作方式。

---

## 二、两个基底仓库（能力原材料）

### 2.1 academic-research-skills（Imbad0202，33K stars）

4 个 Claude Code skill，流水线串联：

| Skill | 角色 | 核心机制 |
|-------|------|----------|
| `deep-research` | 文献调研 | 4 模式（socratic/quick/lit-review/systematic-review），PRISMA，corpus-first flow |
| `academic-paper` | 论文写作 | 12-agent pipeline，10 种 mode，6 种论文类型，5 种引文格式，双语文摘 |
| `academic-paper-reviewer` | 模拟审稿 | 7-agent 多视角审稿，0-100 评分，R&R 追踪矩阵 |
| `academic-pipeline` | 全流程编排 | 10 阶段编排器，Material Passport，claim verification，integrity gate |

### 2.2 nature-skills（Yuan1z0825，21K stars）

8+ 个 Codex/Claude Code skill，以 Nature 期刊标准为基准，各自独立：

| Skill | 角色 | 核心机制 |
|-------|------|----------|
| `nature-writing` | 章节草稿 | 从数据/声明/笔记生成各章节 |
| `nature-polishing` | 语句润色 | 草稿打磨到 Nature 语言标准 |
| `nature-figure` | 科研绘图 | Python/R 投稿级图表，figure-contract 驱动 |
| `nature-citation` | 引文检索 | Crossref 检索 → Nature/CNS 过滤 → ENW/RIS/Zotero 导出 |
| `nature-data` | 数据声明 | 数据可用性声明 |
| `nature-reader` | 论文精读 | 全文 Markdown + 中英对照 + 图文对应 |
| `nature-paper2ppt` | PPT 生成 | 论文转 journal club / 答辩 slides |
| `nature-response` | 审稿回复 | major/minor revision 回应 |

---

## 三、第三基础设施：Karpathy LLM Wiki 知识库系统

### 3.1 这是什么

用户已有一套完整运行的 Karpathy 式个人知识库系统（示例位置：`~/public_workplace/rain-knowledge-base/`）。核心隐喻：**Obsidian = IDE，LLM = 程序员，Wiki = 代码库**。它不是"又一个基底仓库"，而是**持久化知识基础设施**——为论文 skill 体系提供长期记忆和知识复利。

### 3.2 架构

```
<知识库名>/
├── claude.md          — Schema 定义 + 规则（每个 KB 独立）
├── .obsidian/         — Obsidian vault 配置
├── raw/               — 源材料（按主题分目录，编号命名，不可修改）
├── wiki/              — AI 维护的结构化 wiki
│   ├── INDEX.md       — 每个主题 + 一行描述
│   ├── concepts/      — 概念文章
│   ├── comparisons/   — 对比分析
│   ├── entities/      — 工具/人物/实体
│   └── synthesis/     — 综合总结
└── output/            — 生成的报告和分析
```

三个核心操作：
- **ingest（摄入）**：一份源材料触及 10-15 个 wiki 页面，编译知识
- **query（查询）**：向 Wiki 提问，答案归档
- **lint（健康检查）**：七维度静态分析与自我修复

### 3.3 Agent 协同模型

- **主编（主 session）**：规划研究大纲、派发研究员、汇总去重、综合整理、写入 wiki
- **研究员（子 agent）**：接收单个子主题 → WebSearch → 评估来源质量 → 深度阅读 → 返回结构化笔记。只报告不写入。

### 3.4 与论文 skill 体系的整合点

KB 系统与论文 Passport 之间存在天然映射：

| KB 概念 | Paper Passport 字段 | 关系 |
|---------|-------------------|------|
| raw/ 中的单篇论文 | `citation_registry` 条目 | 一篇 raw 文章 = 一条 citation，包含 BibTeX |
| wiki/concepts/ 中的概念 | `claims`（概念性声明） | wiki 概念为 claims 提供背景支撑 |
| wiki/synthesis/ 中的综合 | `evidence_map`（文献矩阵） | synthesis 文章直接映射为 evidence_map |
| wiki/INDEX.md | 文献检索入口 | 快速定位相关文献 |
| lint 操作 | paper-verify 的引用核查 | lint 检查 KB 内部一致性，verify 检查论文-KB 一致性 |
| ingest 操作 | paper-research 的文献调研 | 每次 ingest = 一次结构化的文献添加 |

### 3.5 整合后的研究流程

```
传统方式：
  paper-research → web search → 返回文献列表 → 丢弃（无持久化）

整合后：
  paper-research → 5 阶段 KB 工作流：
    Phase 1: 规划（主编分解研究问题为子主题）
    Phase 2: 并行搜索（研究员独立搜索+阅读+提取）
    Phase 3: 深度阅读（已在 Phase 2 完成）
    Phase 4: 主编综合（去重+交叉引用+形成文献矩阵）
    Phase 5: 写入 KB（raw/ + wiki/ + INDEX.md + output/）
  → 产出：结构化的 paper-kb + passport patch（citation_registry + evidence_map + claims）
  → KB 持久化，论文写完后仍可查询、复用、lint
```

---

## 四、核心适配问题：基底仓库 vs Reasonix 的运行哲学差异

两个基底仓库的设计出发点与 Reasonix 存在根本差异：

| 维度 | 基底仓库（Claude/Codex） | Reasonix |
|------|--------------------------|----------|
| 入口模型 | skill 假设被用户直接调用 | 必须先经过路由和模式识别 |
| prompt 策略 | 长 prompt 内嵌完整流程 | router + lazy-load，决策规则和内容分离 |
| 状态管理 | 各自维护隐含状态或单层 passport | 全链路共享状态层 |
| 完成声明 | skill 内部自行完成 | 横向门控 skill 统一验证 |
| 用户节奏 | 倾向一次生成完整稿件 | 分阶段 checkpoint，强制确认关键节点 |
| 输出契约 | 混合格式（MD/DOCX/Bib/report） | 每个 skill 明确输入/输出契约 |

**结论：不是"移植"，而是"吸收能力 + 重写 Reasonix 外壳"。**

---

## 五、四层架构

```
┌─────────────────────────────────────────────────┐
│  Reasonix Orchestration Layer（编排层）          │
│  paper-superpowers / paper-task-modes           │
│  paper-brainstorm / paper-plan / paper-write     │
│  paper-verify / paper-verify-lite / paper-debug  │
│  paper-review / paper-response / paper-finalize  │
│  ─────────────────────────────────────────────  │
│  决策链逻辑、checkpoint、质量门控、模式路由       │
│  这些 skill 握有主线控制权                        │
└──────────────┬──────────────────────────────────┘
               │ 调用（通过适配器）
┌──────────────▼──────────────────────────────────┐
│  Adapter Layer（适配层）                         │
│  research_adapter / writing_adapter              │
│  figure_adapter / citation_adapter               │
│  polish_adapter / review_adapter                 │
│  response_adapter / reader_adapter               │
│  ─────────────────────────────────────────────  │
│  每个 adapter 负责：                              │
│  1. 接受 Paper Passport + task spec              │
│  2. 调用基底仓库的能力模块                        │
│  3. 返回 artifact + passport patch + verif. notes│
│  4. 绝不绕过 paper-superpowers                   │
│  5. 绝不自行声明完成                              │
└──────────────┬──────────────────────────────────┘
               │ 按需加载 / 持久化读写
┌──────────────▼──────────────────────────────────┐
│  Reference Layer（参考层）                        │
│  期刊写作规范 / agent 提示模板 / style guide      │
│  figure contract 规则 / citation 格式定义         │
│  PRISMA checklist / integrity 检查清单            │
│  ─────────────────────────────────────────────  │
│  原仓库中的规则和参考内容，按需读取，              │
│  不常驻上下文，不直接嵌入主 skill                  │
├─────────────────────────────────────────────────┤
│  Knowledge Base Layer（知识库基础设施层）         │
│  paper-kb（KB 生命周期管理）                     │
│  ─────────────────────────────────────────────  │
│  为每篇论文创建独立 KB：                          │
│    raw/    ← 文献原文（不可修改）                 │
│    wiki/   ← AI 维护的结构化知识                  │
│    output/ ← 文献矩阵、研究缺口报告等              │
│  ─────────────────────────────────────────────  │
│  持久化——论文完成后 KB 仍然存在，可查询、复用     │
│  与 Paper Passport 双向映射：                     │
│    raw/ ↔ citation_registry                     │
│    wiki/synthesis/ ↔ evidence_map               │
│    wiki/concepts/ ↔ claims（概念性支撑）          │
│    lint ↔ paper-verify（引用一致性检查）          │
└─────────────────────────────────────────────────┘
```

**关键规则：**
- 编排层 skill 负责"什么时候做什么"——决策、路由、门控
- 适配层负责"怎么做"——将外部能力封装为 Reasonix 兼容的调用
- 参考层负责"依据什么"——规则、风格、标准，lazy-load
- 基底仓库的原始 SKILL.md 不直接出现在 Reasonix skill 中

---

## 六、Paper Passport — 全链路共享状态

这是所有 skill 输入/输出的共享契约。它不是一个建议格式，而是编排层和适配层的硬性接口。

```yaml
# Paper Passport schema
research_question: string
target_journal: string
paper_type: enum(research|article|review|short|letter)
paper_status: enum(ideation|planning|drafting|polishing|review|revision|final)

claims:
  - id: string
    text: string
    section: string
    evidence: [string]
    source_type: enum(user_data|source_claim|inference|material_gap|needs_confirmation)
    status: enum(supported|weak|unverified|contradicted)

evidence_map:
  - id: string
    type: enum(citation|experiment|user_provided|figure|calculation)
    ref: string
    supports: [string]

citation_registry:
  - key: string
    bibtex: string
    support_level: enum(direct|partial|background)
    used_in: [string]

figure_contracts:
  - id: string
    conclusion: string
    archetype: string
    data_source: enum(user_provided|generated|public_dataset)
    status: enum(planned|drafted|polished|final)

section_status:
  introduction: enum(outlined|drafted|polished|verified|final)
  methods: enum(...)
  results: enum(...)
  discussion: enum(...)
  abstract: enum(...)

term_glossary:
  - term: string
    zh: string
    defined_in: string

known_gaps:
  - description: string
    severity: enum(minor|moderate|critical)

review_findings:
  - id: string
    dimension: string
    score: int(0-100)
    comment: string

revision_history:
  - timestamp: string
    trigger: string       # 触发修改的 review_finding ID
    action: string
    affected_sections: [string]
```

**读写权限规则：**

| 字段组 | 可读 | 可写（修改） | 只追加 | 只有编排层可写 |
|--------|------|-------------|--------|---------------|
| research_question / target_journal / paper_type | 全部 | — | — | brainstorm |
| paper_status | 全部 | 编排层 skill | — | 编排层 |
| claims | 全部 | paper-write, paper-plan | — | paper-write |
| evidence_map | 全部 | paper-research, paper-write | — | paper-write |
| citation_registry | 全部 | paper-citation | paper-research | — |
| figure_contracts | 全部 | paper-figure | — | — |
| section_status | 全部 | paper-write | — | paper-write |
| term_glossary | 全部 | paper-polish, paper-write | — | — |
| known_gaps | 全部 | paper-debug, paper-verify | — | paper-write |
| review_findings | 全部 | paper-review | — | — |
| revision_history | 全部 | paper-write, paper-response | 全部 | — |

**原则：** paper-write 不是凭上下文记忆写，而是围绕 passport 改状态。适配层的每个调用必须返回 passport patch，不能只返回文本。

---

## 七、Skill 清单与角色域（18 个）

按角色域分为四类：

### 6.1 路由与模式（2 个）

| Skill | 对标 | 职责 |
|-------|------|------|
| `paper-superpowers` | using-superpowers | 论文行动总路由，强制检查，优先级定义，合理化借口反驳表 |
| `paper-task-modes` | task-modes | 识别 9 种模式，加载行为约束，触发 ponytail 和 verify |

### 6.2 主线控制（3 个）— 握有论文从 idea 到正文的核心推进权

| Skill | 对标 | 职责 |
|-------|------|------|
| `paper-brainstorm` | brainstorming | 8 步强制流程（探索→提问→方案→展示→批准→写文档→自检→审查→过渡到 paper-plan） |
| `paper-plan` | writing-plans | 大纲→章节计划→claim 分配→图表位置→字数预算→自检→输出执行选项 |
| `paper-write` | subagent-driven-development + executing-plans 混合 | **主 session 握笔**：逐章生成草稿 + 外包专业工种 + paper-verify-lite 门控 + 更新 passport |

### 6.3 质量门控（3 个）— 不参与写作，只做判断

| Skill | 对标 | 职责 |
|-------|------|------|
| `paper-verify-lite` | verification-before-completion（轻量版） | 每章后机械检查：claim 支撑/数据一致性/章节目标。阻塞下一章 |
| `paper-verify` | verification-before-completion（完整版） | 全稿验证：引用核查/数据一致性/逻辑链/格式合规/术语一致性/provenance 完整性 |
| `paper-debug` | systematic-debugging | 四阶段论文诊断（症状→模式→假设→修复） |

### 6.4 专业工种（9 个）— 独立技能，被主线或用户直接调用，不握主线

| Skill | 基底能力 | 职责 |
|-------|---------|------|
| `paper-research` | deep-research + KB 5-phase 工作流 | 文献调研：并行搜索→深度阅读→主编综合→写入 paper-kb。产出文献矩阵 + passport patch |
| `paper-kb` | Karpathy LLM Wiki 系统 | KB 生命周期：init（创建论文 KB）/ ingest（摄入文献）/ query（查询 KB）/ lint（健康检查）/ merge（论文完成后合并到用户永久 KB） |
| `paper-polish` | nature-polishing + article-polisher | 语句润色，不改变论证/数据/引用，维护 term_glossary |
| `paper-citation` | nature-citation | 正文提取→Crossref 检索→期刊过滤→ENW/RIS/Zotero 导出 |
| `paper-figure` | nature-figure | figure-contract 驱动，Python/R 双后端 |
| `paper-review` | academic-paper-reviewer | 全稿 7-agent 模拟审稿，写入 passport.review_findings |
| `paper-response` | nature-response | 逐条回复审稿意见，修改稿标注变更 |
| `paper-finalize` | academic-paper formatter | 格式检查/supplementary/cover letter/投稿包导出 |
| `paper-read` | nature-reader | 论文精读（结构提取+中英对照+声明定位） |
| `paper-ppt` | nature-paper2ppt | 论文转 presentation/journal club |

`paper-init` 不算正式 skill。降级为 paper-superpowers 的项目脚手架能力——创建目录骨架和空文件，不参与决策。

### 6.5 横向约束层（2 个，贯穿全过程）

| Skill | 对标 | 职责 |
|-------|------|------|
| `paper-ponytail` | ponytail | 论文版 6 级决策阶梯 + 不可懒清单 |
| `paper-provenance` | （无直接对标，新增） | 5 类标签强制标注 + 扫描 + threshold 报警 |

`paper-provenance` 强制执行 5 类来源标签：`[USER DATA]` `[SOURCE CLAIM]` `[INFERENCE]` `[MATERIAL GAP]` `[NEEDS CONFIRMATION]`。

---

## 八、paper-write 混合执行模型

```
paper-write 逐章推进（主 session 全权控制论证一致性）：

每章执行单元：
  1. 加载上下文（passport + 章节 blueprint + 前一章 + 相关文献/图表契约）
  2. 主 session 生成章节草稿（握笔的是主控，不外包）
  3. 外包给专业工种（可并行，不握笔）：
     ├─ paper-citation（为该章声明检索引用）→ 返回 passport patch
     ├─ paper-polish（语句润色）→ 不改变 claims 和 evidence_map
     └─ paper-figure（相关图表定稿）→ 返回 passport patch
  4. paper-verify-lite 门控（passport claims vs 正文）
     ├─ 通过 → 继续
     └─ 失败 → 阻塞，进入 paper-debug
  5. 用户确认（默认确认级别）
  6. 更新 passport（claims/section_status/term_glossary/citation_registry）
  7. 进入下一章

全局规则：
  - Introduction / Discussion / Abstract 之间的口径一致性由主 session 全权负责
  - 专业工种只做它们被要求的那一件事，不越界修改论证
```

---

## 九、两层审稿机制

| | paper-verify-lite | paper-review |
|---|---|---|
| **时机** | 每章草稿完成后 | 全稿完成后 |
| **检查维度** | claim 支撑 / 数据一致性 / 章节目标 / provenance 标签 | 贡献/创新性/方法/逻辑/局限/可发表性（7 维度） |
| **深度** | 机械检查（当前章 vs passport） | 语义审查（模拟真实审稿人） |
| **输出** | pass/fail + 不通过项列表 | structured review report + scores + revision roadmap |
| **阻塞** | fail 时阻塞下一章 | 不阻塞，写入 passport.review_findings |
| **实现** | paper-verify-lite | 通过 review_adapter 调用 academic-paper-reviewer 7-agent |

---

## 十、用户确认分层

```
强制确认（必须等用户回复，不可自动推进，不可降级）：
  - 研究问题确定
  - 目标期刊选择
  - 论文结构/大纲批准
  - 核心 claims 列表确认
  - 投稿前最终版本确认

默认确认（每节完成时展示，用户可说"继续"快速通过）：
  - Introduction 完成
  - Methods 完成
  - Results 完成
  - Discussion 完成
  - Abstract 完成（全稿后）
  - 模拟审稿报告查看

自动推进（不打断用户，只记录日志）：
  - 语句润色
  - 引用格式统一
  - 图表导出格式调整
  - paper-verify-lite 通过时
  - paper-citation 补充引用时

自动驾驶模式（用户声明后启用）：
  - "每节确认"降级为"每 2-3 节汇总确认"
  - 强制确认点仍然保留，不可降级
  - material_gap 和 needs_confirmation 数量超阈值时自动升格为强制确认
```

---

## 十一、适配层规范（5 条铁律）

1. **每个外部 skill 只能作为 backend capability**，不能绕过 paper-superpowers 被直接调用。
2. **每个 backend capability 必须接受 Paper Passport** 作为输入的一部分，不能只接收纯文本 task。
3. **每次调用必须返回 passport patch**，不能只返回文本 artifact。调用方负责验证 patch 合法性后合并到 passport。
4. **所有完成声明必须经过 paper-verify 或 paper-verify-lite**。适配层禁止自行声明任务完成或输出质量合格。
5. **大型规则文件必须 lazy-load**，通过 Reference Layer 按需读取。不将期刊写作规范、agent 提示模板、style guide 常驻在主 skill 上下文。

---

## 十二、核心决策链（最终版）

```
用户：「帮我写一篇论文」
  │
  ▼
paper-superpowers（总路由）
  │
  ▼
paper-task-modes → 模式=论文写作 → 触发 paper-ponytail
  │
  ▼
paper-brainstorm
  ├─ [可选] paper-research (quick) + paper-kb init 创建论文 KB
  ├─ 逐一提问 → 研究问题/gap/方法/目标期刊
  ├─ 2-3 方案 → [强制确认] 用户选择
  ├─ 写研究设计文档 + passport 初稿
  ├─ 自检 → [强制确认] 用户审查
  └─ 调用 paper-plan
  │
  ▼
paper-plan
  ├─ 章节结构 + claims 分配 + 图表映射 + 字数
  ├─ 自检 → [强制确认] 大纲 + 核心 claims
  └─ 输出大纲 + 更新 passport
  │
  ▼
paper-research（系统性文献调研，可并行于 paper-plan）
  ├─ Phase 1: 主编分解研究问题 → 子主题
  ├─ Phase 2: 并行派发研究员 → 搜索+阅读+提取
  ├─ Phase 3: 深度阅读（已在 Phase 2 完成）
  ├─ Phase 4: 主编综合 → 去重+交叉引用+文献矩阵
  ├─ Phase 5: paper-kb ingest → raw/ + wiki/ + INDEX.md
  └─ 产出 passport patch（citation_registry + evidence_map）
  │
  ▼
paper-write（逐章循环）
  │
  ├─ Introduction:
  │   ├─ 主 session 草稿（可从 paper-kb query 查概念支撑）
  │   ├─ paper-citation（外包）
  │   ├─ paper-polish（外包）
  │   ├─ paper-verify-lite
  │   └─ [默认确认]
  │
  ├─ Methods / Results / Discussion（同上循环）
  │   └─ paper-figure 并行推进
  │
  ├─ Abstract（全稿后，主 session 握笔）
  │
  ▼
paper-review（全稿模拟审稿）→ [默认确认] 查看审稿报告
  │
  ▼
paper-response（如有修改）/ paper-polish（终润）
  │
  ▼
paper-finalize → [强制确认] 投稿
  │
  ▼
paper-kb merge（可选）→ 论文 KB 合并到用户永久知识库
```

---

## 十三、下一步：三份宪法文档

不先写 17 个 skill。先定三份宪法级文档，让所有 skill 在同一个规则框架下运作：

1. **Paper Passport Schema**（完整字段定义 + 读写权限矩阵 + 初始化和迁移规则）
2. **Skill 调用边界规范**（每个 skill 的输入契约/输出契约/调用权限/禁止行为清单/适配层接口签名）
3. **Checkpoint 策略**（四层确认的完整矩阵 + 自动驾驶降级规则 + 每条规则的违反后果 + provenance threshold 定义）

三件定了之后，单个 skill 的 SKILL.md 只需要写「做什么」和「怎么做」。
