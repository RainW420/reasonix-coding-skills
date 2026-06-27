# 论文写作 Skill 体系 — 架构方案终版

> 仿照 Reasonix code skill 决策链结构，在论文写作领域构建同等水平的流程约束和自动化。基底能力来自 academic-research-skills 和 nature-skills，基础设施层来自 Karpathy LLM Wiki 知识库系统。全部经过 Reasonix 适配层封装。

---

## 一、三层能力来源

### 1. Reasonix Code Skill 决策链（架构模板）

核心结构：「总路由 → 模式识别 → 创意规划 → 执行计划 → 分派执行 → 横向约束 → 收尾」

关键设计哲学：
- "先想再做"不可跳过链路 — 层层检查点确保方向正确
- "写完才算做对"的证据主义 — 空口声称完成 = 违规
- "最小代码最大效果"的激进克制 — 6 级决策阶梯
- "测试是代码的生存前提" — TDD 铁律
- "契约优先于实现" — agent 边界必须先建立契约矩阵
- "隔离即安全" — 子代理上下文隔离 + git worktree 物理隔离

### 2. academic-research-skills + nature-skills（能力原材料，作为 backend capability 被适配层调用）

- academic-research-skills（Imbad0202，33K stars）：4 个 Claude Code skill — deep-research（文献调研 4 模式）、academic-paper（12-agent 写作流水线 10 模式）、academic-paper-reviewer（7-agent 审稿）、academic-pipeline（10 阶段编排器 + Material Passport）
- nature-skills（Yuan1z0825，21K stars）：8+ 个 Codex skill — nature-writing（章节草稿）、nature-polishing（语句润色）、nature-figure（科研绘图）、nature-citation（引文检索）、nature-reader（论文精读）、nature-paper2ppt（PPT）、nature-response（审稿回复）、nature-data（数据声明）

**不原样移植。** 两个基底仓库的原始 SKILL.md 设计为 Claude/Codex 自包含长 prompt 模式，与 Reasonix 的约束决策链在入口、上下文策略、状态管理、完成声明、用户节奏和输出契约六个维度上存在根本冲突。方案采用"吸收能力 + 重写 Reasonix 外壳"策略。

### 3. Karpathy LLM Wiki 知识库系统（基础设施层，持久化知识管理）

用户已有完整运行的 KB 系统（示例位置：`~/public_workplace/rain-knowledge-base/`），核心隐喻 Obsidian=IDE, LLM=程序员, Wiki=代码库。三层目录结构 raw/（不可修改源材料）→ wiki/（AI 维护结构化知识：concepts/comparisons/entities/synthesis + INDEX.md）→ output/（报告分析）。三个核心操作：ingest（摄入，一份源材料触及 10-15 wiki 页面）、query（查询并归档答案）、lint（七维度静态分析自我修复）。Agent 协同模式：主编（规划/派发/综合/写入）+ 研究员（搜索/阅读/提取，只报告不写入）。

---

## 二、四层系统架构

```
┌─────────────────────────────────────────────────┐
│  Orchestration Layer（编排层）                   │
│  paper-superpowers / paper-task-modes           │
│  paper-brainstorm / paper-plan / paper-write     │
│  paper-verify / paper-debug / paper-review          │
│  paper-response / paper-finalize                    │
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

## 三、Paper Passport（全链路共享状态）

所有 skill 的硬性输入/输出契约。不是建议格式。

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

citation_registry:              # 引用注册表
  - key: string
    bibtex: string
    support_level: enum(direct|partial|background)
    used_in: [string]

figure_contracts:               # 图表契约
  - id: string
    conclusion: string          # 该图要证明的结论
    archetype: string
    data_source: enum(user_provided|generated|public_dataset)
    status: enum(planned|drafted|polished|final)

section_status:                 # 每章状态
  introduction: enum(outlined|drafted|polished|verified|final)
  methods|results|discussion|abstract: ...

term_glossary:                  # 术语表（锁定跨章节一致性）
  - term: string / zh: string / defined_in: string

known_gaps:                     # 已知缺口（诚实披露）
  - description: string / severity: enum(minor|moderate|critical)

review_findings:                # 审稿发现
  - id: string / dimension: string / score: int(0-100) / comment: string

revision_history:               # 修改轨迹
  - timestamp: string / trigger: string / action: string / affected_sections: [string]
```

**读写权限矩阵（关键字段）：**

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

**关键约束：**
- `paper-research` 只能写 `evidence_candidates` 和追加 `citation_registry`，**不能**直接写入 `evidence_map.supports`——研究员不替论文主线决定哪个证据支撑哪个论点
- `paper-verify` 可以把证据标记为 `weak` / `unverified`，但不改正文 claim
- `paper-review`（模拟审稿）只能写 `review_findings`，**不能直接修改正文**。修改必须通过 `paper-response` 或 `paper-write revision mode` 执行

---

## 四、18 个 Skill 按角色域分类

### 路由与模式（2）

| Skill | 对标 | 职责 |
|-------|------|------|
| `paper-superpowers` | using-superpowers | 论文行动总路由，强制检查，优先级定义，合理化借口反驳表，中国特色路由 |
| `paper-task-modes` | task-modes | 识别 9 种模式（文献调研/论文写作/修改润色/审稿模拟/审稿回复/投稿准备/论文精读/科研绘图/PPT），加载行为约束，前置触发 ponytail |

### 主线控制（3）— 握有论文从 idea 到正文的核心推进权

| Skill | 对标 | 职责 |
|-------|------|------|
| `paper-brainstorm` | brainstorming | 8 步强制流程：探索文献→逐一提问→2-3 方案→展示设计→批准→写文档→自检→审查→过渡到 paper-plan。可选触发 paper-research 摸底 |
| `paper-plan` | writing-plans | 章节结构→claim 分配→图表映射→字数预算→自检→输出大纲+更新 passport |
| `paper-write` | 混合模型 | **主 session 握笔**。逐章：加载上下文→草稿→外包工种→paper-verify lite mode 门控→用户确认→更新 passport。论证一致性由主 session 全权负责 |

### 质量门控（3）— 不参与写作，只做判断。是主线必经阶段

| Skill | 对标 | 职责 |
|-------|------|------|
| `paper-verify` | verification-before-completion | 两种模式：**lite mode**（每章后机械检查：claim 支撑/数据一致性/章节目标/provenance 标签，fail 阻塞下一章）+ **full mode**（全稿验证：引用核查/逻辑链/格式合规/术语一致性/provenance 完整性）。内置 provenance 协议检查 |
| `paper-debug` | systematic-debugging | 四阶段论文诊断：症状识别→模式对比→假设验证→修复（每次改一个根因） |
| `paper-review` | academic-paper-reviewer（门控角色） | 全稿 7-agent 模拟审稿。**只能写 `review_findings`，不能直接修改正文**。修改必须通过 paper-response 或 paper-write revision mode 执行。是主线必经门控阶段，但不握写作权——类似 CI 报告：可以阻止盲目投稿，不能自己改代码 |

### 专业工种（8）— 被主线或用户调用，不握主线控制权

| Skill | 基底能力 | 职责 |
|-------|---------|------|
| `paper-research` | deep-research + KB 5-phase 工作流 | 文献调研：并行搜索→深度阅读→主编综合→写入 KB。产出 `evidence_candidates` + 追加 `citation_registry`。**不绑定 evidence 到 claim** |
| `paper-polish` | nature-polishing + article-polisher | 语句润色，不改变论证/数据/引用，维护 term_glossary |
| `paper-citation` | nature-citation | 正文提取→Crossref 检索→期刊过滤→ENW/RIS/Zotero 导出 |
| `paper-figure` | nature-figure | figure-contract 驱动，Python/R 双后端 |
| `paper-response` | nature-response | 逐条回复审稿意见，修改稿标注变更。消费 `review_findings`，产出正文修订 |
| `paper-finalize` | academic-paper formatter | 格式检查/supplementary/cover letter/投稿包导出 |
| `paper-read` | nature-reader | 论文精读（结构提取+中英对照+声明定位） |
| `paper-ppt` | nature-paper2ppt | 论文转 presentation/journal club |

### 横向约束（2，贯穿全过程）

| Skill | 对标 | 职责 |
|-------|------|------|
| `paper-ponytail` | ponytail | 论文版 6 级决策阶梯：不写→引用→简化→图表替代→合并→才写。不可懒清单：方法可复现/数据真实完整/引用准确/限制诚实披露 |
| `paper-provenance` | 新增（协议型 skill） | 5 类标签强制标注（USER DATA / SOURCE CLAIM / INFERENCE / MATERIAL GAP / NEEDS CONFIRMATION）。协议定义在 paper-provenance，执行挂在 paper-verify、paper-write、paper-citation 中。threshold 超限时升格用户确认级别 |

### 不计入 skill 数量的基础设施

| 组件 | 定位 |
|------|------|
| `paper-kb` | KB 基础设施能力（init/ingest/query/lint/merge），不属于 18 个论文 skill。作为 Knowledge Base Layer 被各 skill 调用 |
| `paper-verify-lite` | paper-verify 的 lite mode，非独立 skill |


---

## 五、paper-write 混合执行模型

```
逐章推进（主 session 全权控制论证一致性）：

每章执行单元：
  1. 加载上下文（passport + blueprint + 前一章 + 相关文献/图表契约）
     → 可从 paper-kb query 获取概念支撑
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
| 阻塞 | fail 时阻塞下一章 | 不阻塞，写入 review_findings |

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
  material_gap + needs_confirmation 超阈值时自动升格为强制确认
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
  ├─ [可选] paper-kb init 创建论文 KB
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
paper-kb merge（可选）→ 论文 KB 合并到用户永久知识库
```

---

## 九、下一步：三份宪法文档

不先写 18 个 skill。先定三份宪法级文档，让所有 skill 在同一个规则框架下运作：

1. **Paper Passport Schema** — 完整字段定义 + 读写权限矩阵 + 初始化和迁移规则
2. **Skill 调用边界规范** — 每个 skill 的输入契约/输出契约/调用权限/禁止行为清单/适配层接口签名
3. **Checkpoint 策略** — 四层确认的完整矩阵 + 自动驾驶降级规则 + provenance threshold 定义 + 违反后果

三件定了之后，单个 skill 的 SKILL.md 只需要写「做什么」和「怎么做」。
