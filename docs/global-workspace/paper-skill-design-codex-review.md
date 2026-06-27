# 论文写作 Skill 体系 — 架构方案（Codex 审核稿）

> 仿照 Reasonix code skill 决策链结构，在论文写作领域构建同等水平的流程约束和自动化。
> 经过三轮迭代（初版 + Reasonix 内部反馈 + Codex 两轮审核），已收敛为可开工的架构定稿。

---

## 一、设计来源与策略

### 三个输入

| 来源 | 角色 | 内容 |
|------|------|------|
| Reasonix Code Skill 决策链 | 架构模板 | 15 个 skill 的完整决策链（路由→模式→规划→执行→门控→收尾），6 条设计哲学 |
| academic-research-skills + nature-skills | 能力原材料 | 12+ 个 Claude/Codex skill，作为 backend capability 被适配层调用 |
| Karpathy LLM Wiki 知识库系统 | 基础设施 | 用户已有的持久化知识管理基础设施（raw/→wiki/→output/，ingest/query/lint 三操作） |

### 核心策略

**不原样移植基底仓库。** 基底仓库的原始 SKILL.md 设计为 Claude/Codex 自包含长 prompt 模式，与 Reasonix 的约束决策链在入口模型、上下文策略、状态管理、完成声明、用户节奏和输出契约六个维度上存在根本冲突。采用"吸收能力 + 重写 Reasonix 外壳 + 适配层封装"策略。

---

## 二、四层系统架构

```
┌─────────────────────────────────────────────────┐
│  Orchestration Layer（编排层）                   │
│  paper-superpowers / paper-task-modes           │
│  paper-brainstorm / paper-plan / paper-write     │
│  paper-verify / paper-debug / paper-review       │
│  paper-response / paper-finalize                │
│  ─────────────────────────────────────────────  │
│  决策链逻辑、checkpoint、质量门控、模式路由       │
│  这些 skill 握有主线控制权                        │
└──────────────┬──────────────────────────────────┘
               │ 调用（通过适配器）
┌──────────────▼──────────────────────────────────┐
│  Adapter Layer（适配层）                         │
│  research_adapter / writing_adapter / ...       │
│  ─────────────────────────────────────────────  │
│  接受 Paper Passport + task spec                 │
│  调用基底仓库能力模块                             │
│  返回 artifact + passport patch + verif. notes   │
│  不绕过编排层，不自行声明完成                     │
└──────────────┬──────────────────────────────────┘
               │ 按需加载              │ 持久化读写
┌──────────────▼──────────┐  ┌─────────▼──────────┐
│  Reference Layer        │  │  Knowledge Base     │
│  期刊规范/agent模板/     │  │  Layer（基础设施）   │
│  style guide/PRISMA...  │  │  paper-kb           │
│  ─────────────────────  │  │  ────────────────   │
│  只读，lazy-load        │  │  raw/ → wiki/ →     │
│  不常驻上下文            │  │  output/            │
│                          │  │  论文完成后KB持久化  │
└─────────────────────────┘  │  可查询/复用/合并    │
                              └────────────────────┘
```

---

## 三、Paper Passport — 全链路共享状态

所有 skill 的硬性输入/输出契约。不是建议格式。violating skill 的调用方拒绝合并其 passport patch。

```yaml
# 核心字段
research_question: string
target_journal: string
paper_type: enum(research|article|review|short|letter)
paper_status: enum(ideation|planning|drafting|polishing|review|revision|final)

claims:                         # 必须标注 source_type
  - id: string
    text: string
    section: string
    evidence: [string]          # paper-write 绑定的支撑证据 ID
    source_type: enum(user_data|source_claim|inference|material_gap|needs_confirmation)
    status: enum(supported|weak|unverified|contradicted)

evidence_candidates:            # 研究员产出的候选证据（待 paper-write 确认绑定）
  - id: string
    type: enum(citation|experiment|user_provided|figure|calculation)
    ref: string
    proposed_for: [string]      # 建议支撑的 claim ID（仅建议，paper-write 最终决定）
    support_level: enum(direct|partial|background)

evidence_map:                   # 证据图谱（paper-write 正式绑定后写入）
  - id: string
    type: enum(citation|experiment|user_provided|figure|calculation)
    ref: string
    supports: [string]          # 正式绑定的 claim ID

citation_registry:
  - key: string / bibtex: string / support_level: enum(direct|partial|background) / used_in: [string]

figure_contracts:
  - id: string / conclusion: string / archetype: string
    data_source: enum(user_provided|generated|public_dataset)
    status: enum(planned|drafted|polished|final)

section_status:
  introduction: enum(outlined|drafted|polished|verified|final)
  methods|results|discussion|abstract: ...

term_glossary:
  - term: string / zh: string / defined_in: string

known_gaps:
  - id: string
    description: string
    severity: enum(minor|moderate|critical)
    status: enum(open|mitigated|accepted|resolved)
    owner: enum(user|paper-write|paper-research|paper-figure|paper-citation)

review_findings:
  - id: string / dimension: string / score: int(0-100) / comment: string

revision_history:
  - timestamp: string / trigger: string / action: string / affected_sections: [string]
```

**读写权限矩阵：**

| 字段组 | 可追加 | 可读写 | 只有 paper-write 可绑定 |
|--------|--------|--------|------------------------|
| claims | — | paper-write（创建/修改） | — |
| evidence_candidates | paper-research（只追加） | — | — |
| evidence_map | — | — | paper-write（从 candidates 中绑定到 claim） |
| section_status | — | paper-write | — |
| citation_registry | paper-research, paper-citation（追加） | — | — |
| figure_contracts | — | paper-figure | — |
| review_findings | — | paper-review | — |
| term_glossary | — | paper-polish, paper-write | — |
| known_gaps | paper-debug（追加） | paper-verify（更新状态） | — |
| revision_history | 全部（追加） | paper-write, paper-response（修改记录） | — |

**三条关键约束：**
- `paper-research` 只能写 `evidence_candidates` 和追加 `citation_registry`，不能直接写入 `evidence_map.supports`——研究员不替论文主线决定哪个证据支撑哪个论点
- `paper-verify` 可以把证据标记为 `weak`/`unverified`，但不改正文 claim
- `paper-review`（模拟审稿）只能写 `review_findings`，不能直接修改正文。修改必须通过 `paper-response` 或 `paper-write revision mode` 执行

---

## 四、18 个 Skill 按角色域分类

### 路由与模式（2）

| Skill | 对标 | 职责 |
|-------|------|------|
| `paper-superpowers` | using-superpowers | 论文行动总路由，强制检查，优先级定义，合理化借口反驳表，中国特色路由 |
| `paper-task-modes` | task-modes | 识别 9 种模式，加载行为约束，前置触发 ponytail |

### 主线控制（3）— 握有论文从 idea 到正文的核心推进权

| Skill | 对标 | 职责 |
|-------|------|------|
| `paper-brainstorm` | brainstorming | 8 步强制流程：探索文献→逐一提问→2-3 方案→展示设计→批准→写文档→自检→审查→过渡到 paper-plan |
| `paper-plan` | writing-plans | 章节结构→claim 分配→图表映射→字数预算→自检→输出大纲+更新 passport |
| `paper-write` | 混合模型 | **主 session 握笔**。逐章：加载上下文→草稿→外包工种→paper-verify lite mode 门控→用户确认→更新 passport |

### 质量门控（3）— 不参与写作，只做判断。是主线必经阶段

| Skill | 对标 | 职责 |
|-------|------|------|
| `paper-verify` | verification-before-completion | 两种模式：**lite mode**（每章后机械检查，fail 阻塞下一章）+ **full mode**（全稿验证）。内置 provenance 协议检查 |
| `paper-debug` | systematic-debugging | 四阶段论文诊断：症状识别→模式对比→假设验证→修复（每次改一个根因） |
| `paper-review` | academic-paper-reviewer（门控角色） | 全稿 7-agent 模拟审稿。只能写 `review_findings`，不能直接修改正文。修改必须通过 paper-response 或 paper-write revision mode。**不阻塞 paper-response/revision，但若出现 critical finding，则阻塞 paper-finalize——是否继续投稿必须经过强制确认** |

### 专业工种（8）— 被主线或用户调用，不握主线控制权

| Skill | 基底能力 | 职责 |
|-------|---------|------|
| `paper-research` | deep-research + KB 5-phase 工作流 | 文献调研。产出 `evidence_candidates` + 追加 `citation_registry`。不绑定 evidence 到 claim |
| `paper-polish` | nature-polishing + article-polisher | 语句润色，不改变论证/数据/引用，维护 term_glossary |
| `paper-citation` | nature-citation | 正文提取→Crossref 检索→期刊过滤→ENW/RIS/Zotero 导出 |
| `paper-figure` | nature-figure | figure-contract 驱动，Python/R 双后端 |
| `paper-response` | nature-response | 消费 `review_findings`，产出 `revision_plan` + `proposed_patch`（不直接改正文）。正式合并正文时由 paper-write revision mode 或主 session 执行。更新 `revision_history` |
| `paper-finalize` | academic-paper formatter | 格式检查/supplementary/cover letter/投稿包导出 |
| `paper-read` | nature-reader | 论文精读（结构提取+中英对照+声明定位） |
| `paper-ppt` | nature-paper2ppt | 论文转 presentation/journal club |

### 横向约束（2，贯穿全过程）

| Skill | 对标 | 职责 |
|-------|------|------|
| `paper-ponytail` | ponytail | 论文版 6 级决策阶梯 + 不可懒清单 |
| `paper-provenance` | 新增（协议型 skill） | 5 类标签强制标注（USER DATA / SOURCE CLAIM / INFERENCE / MATERIAL GAP / NEEDS CONFIRMATION）。协议定义在 paper-provenance，执行挂在 paper-verify、paper-write、paper-citation。threshold 超限时升格用户确认级别 |

### 不计入 skill 数量的基础设施

| 组件 | 定位 |
|------|------|
| `paper-kb` | KB 基础设施能力（init/ingest/query/lint/merge），不属于 18 个论文 skill |
| `paper-verify lite mode` | paper-verify 的 lite mode，非独立 skill |

**合计：路由 2 + 主线 3 + 门控 3 + 工种 8 + 横向 2 = 18 个 skill。**

---

## 五、paper-write 混合执行模型

```
逐章推进（主 session 全权控制论证一致性）：

每章执行单元：
  1. 加载上下文（passport + blueprint + 前一章 + 相关文献/图表契约）
     → 可从 KB query 获取概念支撑
  2. 主 session 生成章节草稿（握笔的是主控，不外包——论证构建权不交出）
  3. 外包给专业工种（可并行，不握笔）：
     ├─ paper-citation → 返回 passport patch
     ├─ paper-polish → 不改变 claims/evidence_map
     └─ paper-figure → 返回 passport patch
  4. paper-verify lite mode 门控
     ├─ 通过 → 继续
     └─ 失败 → 阻塞，进入 paper-debug
  5. 用户确认（默认确认级别，可说"继续"快速通过）
  6. 更新 passport
  7. 进入下一章

全局约束：
  - Introduction / Discussion / Abstract 之间的口径一致性由主 session 全权负责
  - 专业工种只做被要求的那一件事，不越界修改论证
```

---

## 六、两层审稿 + 四层确认

### 审稿两层

| | paper-verify lite mode | paper-review |
|---|---|---|
| 时机 | 每章草稿后 | 全稿完成后 |
| 深度 | 机械检查（当前章 vs passport） | 语义审查（7-agent 模拟真实审稿） |
| 阻塞 | fail 时阻塞下一章 | 不阻塞 paper-response/revision；若有 critical finding，阻塞 paper-finalize，投稿前强制确认 |
| 修改 | — | 只能写 review_findings，不能直接改正文 |

### 用户确认四层

```
强制确认（不可自动推进，不可降级）：
  研究问题 / 目标期刊 / 大纲结构 / 核心 claims / 投稿前最终版本

默认确认（展示结果，用户说"继续"即过）：
  Introduction / Methods / Results / Discussion / Abstract 完成，审稿报告查看

自动推进（不打断用户，记录日志）：
  语句润色 / 引用格式 / 图表导出 / verify lite mode 通过 / 引用补充

自动驾驶模式（用户声明后启用）：
  每节确认→每 2-3 节汇总确认
  强制确认点保留不可降级
  MATERIAL GAP + NEEDS CONFIRMATION 超阈值时自动升格为强制确认
```

---

## 七、适配层 5 条铁律

1. 每个外部 skill 只能作为 backend capability，不能绕过 paper-superpowers
2. 每个 backend capability 必须接受 Paper Passport
3. 每次调用必须返回 passport patch，不能只返回文本
4. 所有完成声明必须经过 paper-verify（lite mode 或 full mode）
5. 大型规则文件必须 lazy-load，通过 Reference Layer 按需读取

---

## 八、核心决策链（完整路径）

```
用户：「帮我写一篇论文」
  │
  ▼
paper-superpowers（总路由）→ paper-task-modes → 触发 paper-ponytail
  │
  ▼
paper-brainstorm
  ├─ [可选] KB init 创建论文 KB
  ├─ [可选] paper-research (quick) 文献摸底
  ├─ 逐一提问 → [强制确认] 研究问题/目标期刊/方案
  ├─ 写研究设计文档 + passport 初稿 → 自检
  └─ [强制确认] 用户审查 → 调用 paper-plan
  │
  ▼
paper-plan
  ├─ 章节结构 + claims + 图表 + 字数 → 自检
  └─ [强制确认] 大纲 + 核心 claims → 更新 passport
  │
  ▼
paper-research（系统性文献调研，可与 paper-plan 并行）
  ├─ Phase 1-5: 主编分解 → 并行派发研究员 → 综合 → KB ingest
  └─ 产出 passport patch（citation_registry + evidence_candidates）
  │
  ▼
paper-write（逐章循环，主 session 握笔）
  ├─ Introduction: 草稿 → paper-citation + paper-polish → verify lite mode → [默认确认]
  ├─ Methods / Results / Discussion（同上）
  │   └─ paper-figure 并行推进
  └─ Abstract（全稿后，主 session 握笔）
  │
  ▼
paper-review（全稿模拟审稿，只写 review_findings，不改正文）→ paper-verify（full mode）→ [默认确认]
  │
  ▼
paper-response（修改）+ paper-polish（终润）
  │
  ▼
paper-finalize → [强制确认] 投稿
  │
  ▼
KB merge（可选）→ 论文 KB 合并到用户永久知识库
```

---

## 九、后续迭代中可考虑的扩展

以下能力在基底仓库中存在，但未纳入 18 个 skill 的首期清单。可在验证核心链后再扩展：

| 能力 | 基底来源 | 建议纳入时机 |
|------|---------|-------------|
| `nature-data`（数据可用性声明） | nature-skills | paper-finalize 的 supplementary 检查项 |
| `academic-pipeline`（全流程编排器 + Material Passport） | academic-research-skills | Paper Passport 验证后，作为 pipeline 层的强化 |
| `paper-init`（项目脚手架） | 新建 | paper-superpowers 的子命令，不独立成 skill |
| 中文论文特殊支持 | article-polisher + chinese-documentation | paper-task-modes 检测到中文后自动启用 |

---

## 十、下一步：三份宪法文档

不先写 18 个 skill。先定三份宪法级文档：

1. **Paper Passport Schema** — 完整字段定义 + 读写权限矩阵 + 初始化和迁移规则 + evidence_candidates → evidence_map 的状态机
2. **Skill 调用边界规范** — 每个 skill 的输入契约/输出契约/调用权限/禁止行为清单/适配层接口签名
3. **Checkpoint 策略** — 四层确认的完整矩阵 + 自动驾驶降级规则 + provenance threshold 定义 + 违反后果

三件定了之后，单个 skill 的 SKILL.md 只需要写「做什么」和「怎么做」。
