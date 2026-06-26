---
name: 07-outline
description: >
  中文项目申请书写作流程第 07 阶段工具：申请书内容架构与体量规划。
  读取申请书模板、06-helm 的整体方案蓝图、05-synthesis 的综合理解与证据账本，
  生成可扩展的申请书大纲、章节体量预算、writing units、证据分配表和逐单元写作状态，
  为 08-section-write 提供可直接执行的写作任务。

  当用户输入 /grant-master:07-outline，或在 grant 工作流中需要生成申请书大纲、根据 helm 蓝图和模板制定写作计划、
  拆解 writing units、规划篇幅预算、初始化写作状态追踪时，使用本 Skill。

  本 Skill 是 06-helm 的下游、08-section-write 的上游。
  它只负责内容架构、体量规划、写作单元拆解和状态初始化，不写申请书正文。
---

# 07-outline：申请书内容架构与体量规划

## 1. 阶段定位

本 Skill 负责中文项目申请书 workflow 的第 07 阶段：**申请书内容架构与体量规划**。

它不是一个简单的“大纲生成器”。它要解决的是：

```text
如何把 helm 已经收敛出的项目方案，转化为一套可写、可扩展、可追踪、可控制篇幅的申请书写作架构。
```

核心职责：

```text
06_helm（helm_report.md + scheme_blueprint.yaml）
  + 05_synthesis（current_view.md + evidence_ledger.yaml）
  + 申请书模板 / requirements
    ↓
07_outline（本 Skill）
  ├── 解析模板，保留官方章节结构
  ├── 将 helm 方案映射到模板栏目
  ├── 生成逻辑大纲 outline_report.md
  ├── 生成章节体量预算 volume_budget.yaml
  ├── 拆解 writing units writing_units.yaml
  ├── 分配上游证据 source_allocation.yaml
  ├── 规划图片与 Codex 生图提示词 figure_plan.yaml
  ├── 规划表格结构 table_plan.yaml
  ├── 规划参考文献标签 citation_plan.yaml
  ├── 初始化逐节/逐单元写作状态 outline_state.yaml
  ├── 生成结构化蓝图 outline_blueprint.yaml
  └── 输出阶段状态 outline_result.yaml
    ↓
08_section_write / 08_unit_write
```

本阶段的根本任务：

```text
不是把目录列出来，而是把申请书拆成大量可执行的写作单元。
每个单元都要有目标字数、论证任务、证据来源、段落槽位和写作边界。
```

后续 `08-section-write` 不应该再重新思考“这一节写什么”，而应按本阶段生成的 writing units 和 source allocation 逐单元扩写。

---

## 2. 工作目录与文件约定

以 Claude Code 当前工作目录作为项目根目录。

不要硬编码任何绝对路径。所有路径都相对当前工作目录。

```text
.
├── topic.md
├── requirements.md                  # 可选，申报要求、页数/字数/格式限制
├── applicant_profile.md             # 可选，申请人/团队基础
├── references/
│   └── Template.docx                # 可选，申请书模板，可能是 .docx/.doc/.md/.txt
├── workflow/
│   ├── 05_synthesis/
│   │   ├── current_view.md
│   │   ├── evidence_ledger.yaml
│   │   └── latest_result.yaml
│   ├── 06_helm/
│   │   ├── helm_report.md
│   │   ├── scheme_blueprint.yaml
│   │   ├── decision_log.md
│   │   └── helm_result.yaml
│   └── 07_outline/
│       ├── outline_report.md
│       ├── volume_budget.yaml
│       ├── writing_units.yaml
│       ├── source_allocation.yaml
│       ├── figure_plan.yaml          # 图片规划 + Codex 生图提示词
│       ├── table_plan.yaml           # 表格规划 + 08 自动生成规则
│       ├── citation_plan.yaml        # 稳定引用 tag + 参考文献条目
│       ├── outline_state.yaml
│       ├── outline_blueprint.yaml
│       ├── context_bundle.yaml       # 全局上下文束——供所有 writer agent 使用
│       └── outline_result.yaml
```

如果 `workflow/07_outline/` 不存在，应创建。

---

## 3. 状态管理边界

`./workflow/proposal_state.yaml` 只属于 `auto` 管理。

本 Skill 绝不读取、修改或创建：

```text
./workflow/proposal_state.yaml
```

本 Skill 只通过 `workflow/07_outline/outline_result.yaml` 向 `auto` 汇报结果。

---

## 4. 输入文件读取规则

本 Skill 有权限读取 01-06 阶段的全部结果，但不应默认泛读所有历史文件。

采用三级读取规则：

```text
L1：主输入，默认必读；
L2：证据索引，默认读取；
L3：原始细节，按需追溯。
```

### 4.1 L1：主输入，默认必读

```text
./workflow/06_helm/scheme_blueprint.yaml
./workflow/06_helm/helm_report.md
./workflow/06_helm/helm_result.yaml
./topic.md
```

申请书模板也属于主输入，但模板缺失时不阻塞：

```text
./references/Template.docx
./references/Template.doc
./references/Template.md
./references/Template.txt
./references/提纲.*
```

如果找不到模板文件，使用本 Skill 内置的通用申请书结构作为骨架，并在 `outline_result.yaml` 中记录：

```yaml
template_status: "fallback_builtin_outline"
```

如果缺少 `scheme_blueprint.yaml` 或 `helm_report.md`，应阻塞，提示先完成 `06-helm`。

如果缺少 `topic.md`，应阻塞，提示先完成 `01-topic`。

---

### 4.2 L2：证据索引，默认读取

如果存在，每次调用应读取：

```text
./workflow/05_synthesis/current_view.md
./workflow/05_synthesis/evidence_ledger.yaml
./workflow/05_synthesis/latest_result.yaml
./workflow/04_paper_digest/paper_index.yaml
./workflow/06_helm/decision_log.md
./requirements.md
./applicant_profile.md
./CLAUDE.md
```

用途：

- `current_view.md`：帮助设置研究现状、gap 表达和背景脉络；
- `evidence_ledger.yaml`：为每个 writing unit 分配可用证据；
- `paper_index.yaml`：帮助组织国内外研究现状章节；
- `decision_log.md`：避免把已放弃方向写入主线；
- `requirements.md`：决定页数、字数、模板约束、项目类型；
- `applicant_profile.md`：用于研究基础、可行性和团队基础章节；
- `CLAUDE.md`：读取项目级写作规则。

---

### 4.3 L3：原始细节，按需追溯

不要一开始全部读取 L3。

只有在以下情况才追溯：

1. 为某个 writing unit 需要具体论文数据；
2. `evidence_ledger.yaml` 中某个 claim 的摘要不足；
3. 要为“国内外研究现状”分配代表性论文；
4. 要为“研究基础/可行性”查找已有材料；
5. 要确认某个 conclusion 是否可用于申请书正文。

可追溯文件池：

```text
workflow/04_paper_digest/round_XX/reports/{batch_id}/papers/*.md
workflow/04_paper_digest/round_XX/digest_report.md
workflow/05_synthesis/round_XX/synthesis_report.md
workflow/03_academic_search/round_XX/candidate_papers.md
workflow/03_academic_search/round_XX/search_summary.md
workflow/02_literature_plan/long_plan.yaml
workflow/02_literature_plan/round_XX/round_goal.md
workflow/01_topic_card.md
```

追溯原则：

```text
先从 evidence_ledger.yaml 或 paper_index.yaml 找索引；
再读取对应 paper report 或 digest_report；
不要绕过 synthesis 直接从 academic-search 候选论文形成申请书依据；
如果 helm 与 synthesis 不一致，优先使用 helm 的 scheme_blueprint，并在 outline_result.yaml 中记录差异。
```

---

## 5. 职责边界

### 本 Skill 可以做

1. 读取并解析申请书模板；
2. 保留模板官方栏目结构；
3. 根据 helm 蓝图生成申请书逻辑大纲；
4. 根据目标页数/字数生成章节体量预算；
5. 将章节拆成 writing units；
6. 为每个 writing unit 设置目标字数、目标页数、写作目的、论证要素、段落槽位；
7. 为每个 writing unit 分配 synthesis / evidence_ledger / paper reports / helm 中的材料来源；
8. 规划全书图片、统一图片风格，并为每张图生成完整 Codex 生图提示词；
9. 规划全书表格结构、数据来源和 08 自动生成规则；
10. 为可进入正文的论文来源生成稳定引用 tag 和参考文献条目；
11. 初始化逐节、逐单元状态追踪；
12. 标记 blocked 或材料不足的单元；
13. 生成供 `08-section-write` 直接执行的写作任务。

### 本 Skill 不允许做

1. 不写申请书正文；
2. 不执行 section-write 或 unit-write；
3. 不继续调研；
4. 不精读论文；
5. 不重新做 synthesis；
6. 不重新做 helm 方案收敛；
7. 不修改 06-helm 的任何输出；
8. 不读取、修改或创建 `./workflow/proposal_state.yaml`；
9. 不将 helm decision_log 中标注为 `dropped` 的方向写入主线；
10. 不直接使用 academic-search 候选论文作为申请书依据，除非该论文已进入 paper_digest 或 evidence_ledger；
11. 不编造论文、数据、结论、项目基础或团队成果。

---

## 6. 核心设计原则：outline 必须包含体量规划

本 Skill 不得只生成章节级标题。

必须同时完成：

```text
逻辑大纲（完整标题树，最深 5 级，叶结点标注）
+ 章节体量预算（section tree 递归，与大纲同构）
+ writing units 拆解（叶结点 → units，1 个叶结点 ≥ 1 个 unit）
+ 证据分配
+ 图片规划（位置、风格、提示词、正文占位规则）
+ 表格规划（位置、列结构、数据来源、08 自动生成规则）
+ 引用规划（稳定 tag、可引用 claim、参考文献条目）
+ 段落槽位设计
+ heading_level 标注
+ 写作状态初始化（section tree + unit states + section_index）
```

原因：

```text
如果 outline 只固定章节目录，后续 content-plan 只能在固定大纲里硬塞字数；
这样会导致内容扩展困难，section-write 只能扩写已有标题。
```

所以本 Skill 的输出必须让后续写作自然变长，而不是靠空话堆字数。

---

## 7. 模板解析与章节骨架规则

### 7.1 模板优先级

优先读取 `./references/` 中的模板文件：

```text
Template.docx
Template.doc
Template.md
Template.txt
提纲.*
```

若有 `.docx`，可用 pandoc 转换为 markdown：

```bash
pandoc ./references/Template.docx -t markdown --wrap=none -o /tmp/template_outline.md
```

如果 pandoc 不可用或模板无法解析，不阻塞，回退到内置申请书提纲。

### 7.2 标题保留规则

如果模板明确提供官方标题：

1. 一级、二级标题必须保留原文；
2. 不得改写官方固定标题；
3. 不得删除模板中的必写栏目；
4. 可以在模板标题下添加三级及更深标题，用于增强内容组织；
5. 添加标题必须服务 helm 主线和写作体量，不得为了凑层级而机械拆分。

### 7.3 大纲层级与 writing unit 的挂载关系

本 Skill 应生成**完整的大纲标题树**，最深到五级。**每个 section（不论是否叶结点）至少挂载 1 个 writing unit**。

层级关系：

```text
section tree（L1 → L2 → ... → L5，最深 5 级）
  │
  ├── 每个 section（不论叶/非叶）至少 1 个 unit
  │     │
  │     ├── 非叶结点 unit（section_intro）：1 个
  │     │     ├── 需要 intro 段落：写标题 + 简短开篇（150-500 字）
  │     │     └── 不需要 intro 段落：只写标题（heading only），内容由子级 section 直接开始
  │     │
  │     └── 叶结点 unit：1+ 个
  │           └── paragraph slots（每个 unit 4-8 个）
```

关键规则：

1. **每个 section 至少 1 个 unit**——不论叶结点还是非叶结点。这使得标题层级完整，不会因跳过非叶结点而丢失标题。
2. **非叶结点 unit（section_intro）**：每个非叶结点恰好 1 个 unit。该 unit 的 `needs_intro_paragraph` 决定行为：
   - `true`：该 section 需要在子内容展开前有一段总述性文字（如"1.2 背景及研究意义"的开篇）。unit 写该级标题 + 简短开篇段落（150-500 字）。
   - `false`：该 section 只是一个组织性标题（如"1. 项目介绍"），不需要独立开篇文字。unit 只写该级标题（heading only），正文内容由第一个子级 section 的 unit 直接开始。
3. **叶结点 unit**：1 个叶结点 ≥ 1 个 unit。内容简单的叶结点 1 个 unit 即可；内容复杂（如"三层 QoS 隔离"涉及多个子主题）应拆成多个 unit。
4. **unit ID 命名规范**：`{section_id}-U{NNN}`。例如 section `S01.2` 的 unit 为 `S01.2-U001`（非叶 intro unit）；叶结点 `S01.2.1` 的 unit 为 `S01.2.1-U001`。
5. **heading_number**：每个 section 和 unit 应携带 `heading_number`（由 `section_id` 去掉前缀 "S" 得到，如 `S01.2.1` → `1.2.1`）。它不作为 08-section-write 的标题输出依据（08 输出干净标题），而是给 09-assemble 组装时注入编号使用。
6. **heading_level**：等于该 unit 所属 section 的 `level`。
7. **heading_output_rule**：每个 section 的第 1 个 unit 写该级标题（含编号）。同一 section 的后续 unit 以正文段落衔接，不再重复同级标题。
8. **非叶结点 unit 的字数不计入体量预算的主体论证部分**——它们只是结构性开销（标题行 + 可选简短开篇），不在 volume_budget 中与叶结点争夺篇幅。

示例——完整的 section tree + units：

```text
S01（项目介绍，L1，非叶）
  └── S01-U001: section_intro（needs_intro_paragraph=false → heading only）

  S01.2（背景及研究意义，L2，非叶）
    └── S01.2-U001: section_intro（needs_intro_paragraph=true → 标题 + 开篇概述）

    S01.2.1（应用背景，L3，叶）
      └── S01.2.1-U001（heading_level=3）

    S01.2.2（技术背景，L3，叶）
      └── S01.2.2-U001（heading_level=3）

  S01.4（研究方法，L2，非叶）
    └── S01.4-U001: section_intro（needs_intro_paragraph=true → 标题 + 总体思路开篇）

    S01.4.4（三层QoS隔离，L3，非叶）
      └── S01.4.4-U001: section_intro（needs_intro_paragraph=true → 标题 + QoS隔离总述）

      S01.4.4.1（硬件VL/DSCP层，L4，叶）
        └── S01.4.4.1-U001（heading_level=4）

      S01.4.4.2（软件整形层，L4，叶）
        └── S01.4.4.2-U001（heading_level=4）

      S01.4.4.3（PFC-free数据面层，L4，叶）
        └── S01.4.4.3-U001（heading_level=4）
```

### 7.3.1 needs_intro_paragraph 判断标准

非叶结点 unit 是否需要 intro 段落，按以下标准判断：

| 情况 | needs_intro_paragraph | 理由 |
|---|---|---|
| 该 section 是模板官方二级标题（如"背景及研究意义""研究方法"） | `true` | 官方标题下通常需要一段文字说明本节的组织逻辑 |
| 该 section 的 children 之间有逻辑递进关系需要交代 | `true` | 避免子内容直接堆砌，缺乏衔接 |
| 该 section 仅为纯组织性标题（如"项目介绍"），内容完全由子级承载 | `false` | 写标题即可，开篇文字会显得冗余 |
| 该 section 是 L4/L5 深层非叶结点，仅用于分组同类内容 | `false`（通常） | 深层标题本身已足够具体，不需额外开篇 |
| 该 section 在模板中是形式性栏目（如"人力、设备等投入及项目预算"） | `false`（通常） | 以表格填报为主，不需文字开篇 |

**原则**：不为了凑 intro 而写空话；但也不能让子内容"从天而降"缺少引导。

---

## 8. 体量规划规则

本 Skill 必须生成：

```text
workflow/07_outline/volume_budget.yaml
```

### 8.1 目标篇幅来源

篇幅目标按以下顺序确定：

1. `requirements.md` 或模板中明确写出的页数/字数要求；
2. 用户在当前指令中指定的页数/字数；
3. 项目类型默认值；
4. 如果都没有，使用“中等详尽申请书”默认策略，并标记需要用户确认。

默认策略：

```text
NSFC / 青年类：20-30 页，约 15000-25000 字；
重点项目 / 重大项目：50-100 页，约 40000-80000 字；
工程方案 / 标书：100 页以上，按 requirements 分卷规划；
未知类型：40 页，约 30000 字，需用户确认。
```

### 8.2 体量分配原则

篇幅不平均分配，而按论证权重分配。

优先给：

1. 立项依据 / 背景 / 研究现状；
2. 研究内容；
3. 技术路线 / 研究方案；
4. 验证计划 / 考核指标；
5. 创新点；
6. 研究基础。

不应给太多篇幅给：

1. 过泛的背景；
2. 被 helm 降级为 background_only 的方向；
3. dropped 方向；
4. 模板中形式性栏目。

---

## 9. writing units 拆解规则

本 Skill 必须生成：

```text
workflow/07_outline/writing_units.yaml
```

writing unit 是后续写作的最小执行单元。

### 9.1 每个 writing unit 必须包含

1. `unit_id`（格式：`{section_id}-U{NNN}`，如 `S01.2-U001`、`S01.2.1-U001`）
2. `section_id`（**指向所属 section ID**，不论叶/非叶）
3. `heading_level`（该 unit 在输出文档中的标题级别，等于所属 section 的 level）
4. `is_first_unit_of_section`（是否为所属 section 的第 1 个 unit——决定是否写标题）
5. `unit_type`（`section_intro` / `background` / `related_work` / `gap` / `objective` / `content` / `route` / `innovation` / `validation` / `foundation` / `other`）
6. `needs_intro_paragraph`（**仅 `section_intro` 类型需要**：`true`=写标题+简短开篇段落，`false`=只写标题）
7. `title`
8. `target_words`（`section_intro` 且 `needs_intro_paragraph=false` 时可为 0）
9. `target_pages`
10. `purpose`
11. `role_in_document`
12. `required_elements`
13. `paragraph_slots`（`section_intro` 且 `needs_intro_paragraph=false` 时为空数组）
14. `sources`
15. `evidence_claims`
16. `avoid`
17. `output_file`
18. `status`

### 9.2 单元粒度

推荐粒度（按 unit 类型区分）：

**section_intro unit（非叶结点）：**
```text
needs_intro_paragraph=true：150-500 字（开篇段落，不深入内容细节）
needs_intro_paragraph=false：0 字（仅写标题行，不生成正文段落）
```

**叶结点 unit：**
```text
普通申请书：每个 unit 600-1200 字；
长申请书：每个 unit 1000-1800 字；
标书：每个 unit 500-1500 字，按需求条款或响应项拆分。
```

如果一个叶结点的目标字数超过 2000 字，应将其拆成多个 writing units（共享同一 section ID 和 heading_level）。

叶结点拆分决策：

```text
叶结点目标字数 ≤ 1200 字 → 1 个 unit（叶结点标题 = unit 标题）
叶结点目标字数 1200-2000 字 → 1-2 个 unit（按子主题拆分）
叶结点目标字数 > 2000 字 → 考虑将叶结点提升为非叶，下挂更深层叶结点
```

### 9.3 非背景轮次严禁发散

writing units 必须围绕 helm 选择的主线展开。

不要把 synthesis 中所有方向都拆成正文单元。

被 helm 标为：

```text
dropped
```

的方向不得成为主线 unit。

被 helm 标为：

```text
background_only
```

的方向只能成为背景或研究现状中的简短 unit。

---

## 10. 段落槽位规则

每个 writing unit 必须包含 `paragraph_slots`。

段落槽位用于约束后续写作的展开方式，避免输出过短或泛泛而谈。

每个 unit 通常包含 4-8 个段落槽位，例如：

```text
P1：引入本单元问题；
P2：解释背景或已有方法；
P3：分析核心挑战；
P4：提出本项目思路；
P5：说明机制/模块/流程；
P6：说明验证方式；
P7：小结并衔接下一节。
```

每个 slot 应包含：

1. `slot_id`
2. `role`
3. `target_words`
4. `must_include`
5. `source_hints`
6. `avoid`

---

## 11. source allocation 规则

本 Skill 必须生成：

```text
workflow/07_outline/source_allocation.yaml
```

它为每个 writing unit 分配上游材料。

来源可以包括：

```text
workflow/06_helm/scheme_blueprint.yaml
workflow/06_helm/helm_report.md
workflow/06_helm/decision_log.md
workflow/05_synthesis/current_view.md
workflow/05_synthesis/evidence_ledger.yaml
workflow/04_paper_digest/paper_index.yaml
workflow/04_paper_digest/round_XX/...
topic.md
requirements.md
applicant_profile.md
```

每个 unit 至少应有：

1. helm 来源；
2. synthesis/current_view 来源；
3. evidence_claims，若该 unit 涉及研究现状、gap、baseline、指标或创新性；
4. paper sources，若该 unit 需要引用论文；
5. avoid 列表。

如果某个核心 unit 找不到足够证据，应标记：

```yaml
status: "blocked"
blocked_reason: "缺少支撑证据"
```

---

## 12. figure_plan.yaml 规则

本 Skill 必须生成：

```text
workflow/07_outline/figure_plan.yaml
```

`figure_plan.yaml` 是全书图片规划的唯一权威来源，只规划最终需要用户手动生成、替换或绘制的图片。表格不进入 `figure_plan.yaml`，表格由 `table_plan.yaml` 规划，并由 `08-section-write` / `grant-writer` 自动生成 Markdown 表格。

### 12.1 图片规划模式

图片规划采用“两阶段提示词”模式：

```text
07-outline：生成 prompt_base（完整、可直接用于 Codex 生图）
08-section-write / grant-writer：必要时生成 prompt_final（只做正文贴合，不改核心功能）
```

职责边界：

1. `07-outline` 决定全书需要哪些图、每张图服务哪个论证功能、放在哪个 section / unit、是否必需、图与图之间的风格统一规则；
2. `07-outline` 为每张图生成完整 `codex_prompt_base`，提示词必须达到可以直接复制给 Codex 生图的精细度；
3. `08-section-write` 不得从零发明新图；只能消费 `figure_plan.yaml`，把待插入图片传给 writer；
4. `grant-writer` 可按正文实际语境生成 `codex_prompt_final`，但不得改变 `figure_id`、图片类型、核心元素、论证功能和全局风格；
5. 如果正文写作发现确实需要新增或删除图，writer 只能在 manifest 中提出 `figure_plan_change_request`，不得自行新增正式图号。

### 12.2 必须规划的图片类型

根据项目类型、模板篇幅和论证复杂度，优先规划：

| 图片类型 | 典型位置 | 作用 |
|---|---|---|
| 概念总览图 | 立项依据或研究内容开头 | 一图说明问题、方法、目标之间的逻辑 |
| 研究内容关系图 | 研究内容章节开头 | 展示 2-4 项研究内容之间的递进、并行支撑或闭环关系 |
| 技术路线图 | 研究方案 / 技术路线章节 | 展示输入、方法模块、中间产物、输出和验证闭环 |
| 系统架构图 | 研究方案核心模块处 | 展示模块边界、数据流、控制流和接口关系 |
| 验证流程图 / 指标矩阵 | 验证方案章节 | 展示 baseline、场景、指标和预期判据 |
| 研究基础图 | 研究基础章节 | 汇总前期成果、已有平台、数据或代表性结果 |
| 年度计划图 | 年度计划章节 | 展示任务时序、里程碑和交付物 |

普通 NSFC/青年类申请书建议 5-8 张图片；面上项目建议 8-12 张；重点或重大类项目建议 10-15 张。若用户模板或页数更短，应压缩到最能支撑评审理解的核心图。

### 12.3 图片风格统一规则

所有图必须遵守统一视觉风格：

1. 面向中文项目申请书，风格应专业、清晰、克制，不使用卡通、赛博、海报、营销页、过度立体或强装饰风格；
2. 优先采用白底或浅底、细线条、清晰箭头、有限色彩分组，适合 Word/PDF 打印；
3. 同一申请书内保持统一配色、字体和符号体系；
4. 中文标签应简短，避免把大段正文塞进图内；
5. 图内元素不超过评审可快速理解的复杂度：概念图 5-7 个核心元素，技术路线图可分层但不得拥挤；
6. 尽量使用矢量感流程图、架构图、信息图表达；除非研究对象本身需要，不生成照片质感或写实图片；
7. 不从论文 PDF 截图，不复刻已发表论文图，不使用无法确认版权的图片元素；
8. 默认输出应适合 300 DPI 插图，横向图优先 16:9 或 A4 横向比例，复杂图可指定分区布局。

### 12.4 Codex 生图提示词精细度

每个 `codex_prompt_base` 必须包含以下信息，缺一不可：

1. **图的任务**：这张图要帮助评审理解什么论证；
2. **图的类型**：概念图、技术路线图、系统架构图、流程图、矩阵图等；
3. **画布与比例**：横向/纵向，建议比例，是否适合 A4/Word；
4. **布局结构**：从左到右、从上到下、三层结构、闭环结构、泳道结构、中心辐射结构等；
5. **核心元素**：每个模块、节点、箭头、输入、输出、反馈环、验证模块；
6. **中文文字**：图内建议出现的中文标签，要求短语化；
7. **视觉风格**：配色、线条、字体、留白、专业程度；
8. **约束与禁忌**：不要卡通、不要照片、不要英文大段文字、不要装饰背景、不要过度拥挤；
9. **交付期待**：清晰可读、可插入申请书、可后续手工微调。

提示词应具体到“生成什么”和“不要生成什么”，不要只写“生成一张技术路线图”。如果图内文字很多，应在提示词中要求“保留可编辑流程图风格，文字短标签，避免长句”。

### 12.5 figure_plan.yaml 字段

每张图至少包含：

```yaml
figure_id: "F01"
kind: "figure"
type: "technical_route_diagram"
required: true
target:
  section_id: "S02.4"
  unit_id: "S02.4-U001"
  placement_hint: "在首次概述总体技术路线后插入"
title: "项目总体技术路线图"
caption_draft: "图1 项目总体技术路线图"
argument_role: "说明研究内容、关键模块与验证闭环之间的关系"
source_basis:
  helm: ["workflow/06_helm/scheme_blueprint.yaml"]
  synthesis: []
  evidence_claims: []
content_spec:
  must_include: []
  optional_include: []
  must_not_include: []
visual_style:
  canvas: "16:9 横向，适合插入 Word"
  palette: "白底，蓝绿灰为主，橙色只用于突出关键反馈环"
  typography: "中文黑体风格，短标签，字号清晰"
  layout: "左到右流程 + 底部验证闭环"
codex_prompt_base: |
  生成一张中文科研项目申请书风格的技术路线图……
writer_refinement_policy:
  allow_prompt_final: true
  allowed_changes:
    - "根据正文最终术语微调图内中文标签"
    - "根据插入位置补充图题和图注中的关键词"
  forbidden_changes:
    - "改变图片类型"
    - "删除必含模块"
    - "改变全书统一风格"
status: "planned"                    # planned / needs_user_confirmation / optional / blocked
```

### 12.6 与其他输出的关系

1. `writing_units.yaml` 中涉及图片的 unit，应在 unit 蓝图中添加 `figure_refs: ["F01"]`；
2. `source_allocation.yaml` 的 `figures` 应引用 `figure_id`，不要重复长提示词；
3. `outline_report.md` 的“图片规划”应给人看，列出图号、位置、类型、内容和是否必需；
4. `outline_blueprint.yaml` 应汇总 `figure_plan: "workflow/07_outline/figure_plan.yaml"`；
5. `outline_result.yaml` 应记录 `figure_plan` 输出路径和图片数量。

---

## 13. table_plan.yaml 规则

本 Skill 必须生成：

```text
workflow/07_outline/table_plan.yaml
```

`table_plan.yaml` 是全书表格规划的唯一权威来源。表格不是图片，不需要 Codex 生图提示词；表格是申请书正文内容的一部分，由 `08-section-write` / `grant-writer` 根据 `table_plan.yaml` 自动生成 Markdown 表格，并随 unit 正文进入后续组装。

### 13.1 表格规划模式

表格采用“07 规划、08 生成”的模式：

```text
07-outline：规划表格位置、标题、列结构、行结构、数据来源、填充规则
08-section-write / grant-writer：按 table_specs 自动生成 Markdown 表格正文
```

职责边界：

1. `07-outline` 决定哪些内容应该用表格呈现，而不是大段文字；
2. `07-outline` 为每张表设置 `table_id`、表号、目标 unit、插入位置、列结构、行结构、数据来源和缺失值处理规则；
3. `08-section-write` 必须读取 `table_plan.yaml`，把本批次相关表格作为 `table_specs[]` 写入 batch instruction；
4. `grant-writer` 必须在对应 unit 中直接生成 Markdown 表格，不留“表格待补”占位，除非 `table_plan.status=blocked` 或关键数据缺失；
5. writer 不得从零新增正式表号；若发现需要新增表格，只能在 manifest 中提出 `table_plan_change_request`。

### 13.2 适合表格呈现的内容

优先规划以下表格：

| 表格类型 | 典型位置 | 作用 |
|---|---|---|
| 研究内容分解表 | 研究内容章节 | 对齐研究内容、目标、方法、产出 |
| 技术路线模块表 | 研究方案章节 | 对齐模块、输入、方法、输出、验证 |
| 创新点对照表 | 特色与创新章节 | 对齐现有方法不足、本项目创新、验证方式 |
| 验证指标表 | 验证方案章节 | 对齐任务、baseline、指标、判据、数据来源 |
| 年度计划表 | 年度计划章节 | 对齐年份、任务、里程碑、成果 |
| 研究基础表 | 研究基础章节 | 汇总已有成果、平台、数据、论文或项目基础 |
| 风险与对策表 | 可行性分析章节 | 对齐风险、影响、监测指标、应对措施 |

原则：能让评审快速比较、扫描、核对的内容，优先用表格；需要解释机制和逻辑递进的内容，不要用表格替代正文论证。

### 13.3 表格设计规则

1. 表格必须有明确论证功能，不能为了显得丰富而造表；
2. 列数通常 3-6 列，避免超宽表；长申请书或标书可适当扩展；
3. 每个表格必须有 `columns[]`，每列含 `name`、`purpose`、`source`、`required`；
4. 行结构应可枚举，优先来自 helm 的研究内容、模块、验证任务、年度计划或 applicant_profile 的成果；
5. 表格内容必须来自 `scheme_blueprint.yaml`、`helm_report.md`、`current_view.md`、`evidence_ledger.yaml`、`applicant_profile.md` 或 unit 蓝图，不得编造数据；
6. 数据缺失时，优先写“待补充”或在 manifest 中 warning，不得虚构数字；
7. Markdown 表格前后必须有正文引导和解释，表格不能独立悬空；
8. 表格之后必须有 1-2 句总结，指出评审应从表中看出什么。

### 13.4 table_plan.yaml 字段

每张表至少包含：

```yaml
table_id: "T01"
table_no: 1
type: "validation_metrics_table"
required: true
status: "planned"                    # planned / needs_user_confirmation / optional / blocked
target:
  section_id: "S02.5"
  unit_id: "S02.5-U001"
  placement_hint: "在说明验证方案总体设计后插入"
  placement_after_slot: "P3"
title: "验证任务与评价指标表"
caption_draft: "表1 验证任务与评价指标"
argument_role: "说明各研究任务如何被可观测指标验证"
source_basis:
  helm: ["workflow/06_helm/scheme_blueprint.yaml"]
  synthesis: []
  evidence_claims: []
  applicant_profile: []
columns:
  - name: "验证任务"
    purpose: "对应研究内容或技术模块"
    source: "scheme_blueprint.validation_plan"
    required: true
  - name: "评价指标"
    purpose: "说明可量化评价方式"
    source: "scheme_blueprint.validation_plan.metrics"
    required: true
rows:
  mode: "from_research_contents"       # explicit / from_research_contents / from_modules / from_validation_tasks / from_years
  expected_count: 3
  explicit_items: []
fill_rules:
  missing_value: "待补充"
  allow_estimated_text: false
  numeric_values_must_have_source: true
markdown_rules:
  generate_visible_table: true
  table_format: "markdown"
  require_intro_sentence: true
  require_followup_sentence: true
```

### 13.5 与其他输出的关系

1. `writing_units.yaml` 中涉及表格的 unit，应在 unit 蓝图中添加 `table_refs: ["T01"]`；
2. `source_allocation.yaml` 的 `tables` 应引用 `table_id`，不要重复完整表格设计；
3. `outline_report.md` 的“表格规划”应给人看，列出表号、位置、类型、内容和是否必需；
4. `outline_blueprint.yaml` 应汇总 `table_plan: "workflow/07_outline/table_plan.yaml"`；
5. `outline_result.yaml` 应记录 `table_plan` 输出路径和表格数量；
6. `08-section-write` 必须把相关 `table_specs[]` 传给 writer，writer 必须自动生成表格正文。

---

## 14. citation_plan.yaml 规则

本 Skill 必须生成：

```text
workflow/07_outline/citation_plan.yaml
```

`citation_plan.yaml` 是全书引用标签和参考文献条目的唯一权威来源。它不使用 `[1]`、`[2]` 这类数字编号，因为数字编号只有在 09-assemble 合并全文并知道首次出现顺序后才有意义。

### 14.1 引用规划模式

引用采用“07 生成 tag、08 使用 tag、09 替换编号”的模式：

```text
07-outline：为每篇可引用文献生成稳定 citation tag 和格式化参考文献条目
08-section-write / grant-writer：正文中只写 {{cite:tag}}，不写 [1]/[2]
09-assemble：按正文首次出现顺序把 tag 替换为 [1][2][3]，并生成参考文献列表
```

职责边界：

1. `07-outline` 从 `source_allocation.yaml` 的 `paper_sources`、`evidence_claims` 和 `evidence_ledger.yaml` 中提取会进入正文的论文；
2. `07-outline` 为每篇论文生成稳定、可读、语义化的 `tag`，如 `vaswani2017attention`、`devlin2019bert`；
3. `07-outline` 生成 `reference_text`，但其中不得带最终编号；
4. `08-section-write` 必须把相关 citation specs 传给 writer；
5. `grant-writer` 只能使用 `citation_plan.yaml` 中已有 tag，不能临时发明 tag，不能写数字编号；
6. `09-assemble` 扫描 `{{cite:tag}}`，按首次出现顺序编号，并把使用到的参考文献列表插入到背景/研究现状之后。

### 14.2 tag 生成规则

tag 必须稳定、可读、跨 unit 复用：

```text
{first_author_lastname}{year}{first_meaningful_title_word}
```

示例：

```text
{{cite:vaswani2017attention}}
{{cite:devlin2019bert}}
{{cite:dao2022flashattention}}
{{cite:zhang2024llmserving}}
```

规则：

1. tag 全小写，只使用 `a-z0-9_`；
2. 优先使用第一作者姓氏 + 年份 + 标题第一个有意义实词；
3. 同一 tag 冲突时追加短后缀，如 `wang2023scheduling_a`；
4. 中文作者可使用拼音姓氏；无法可靠转写时使用 `paper_id` 兜底，但应尽量保留年份和标题关键词；
5. tag 不得使用 `CIT001`、`P001`、`ref1` 这类写作时无语义的编号。

### 14.3 citation_plan.yaml 字段

每条引用至少包含：

```yaml
citations:
  - tag: "vaswani2017attention"
    paper_id: "P001"
    title: "Attention Is All You Need"
    authors: "Vaswani et al."
    year: 2017
    venue: "NeurIPS"
    doi: ""
    url: ""
    reference_text: "Vaswani A, Shazeer N, Parmar N, et al. Attention Is All You Need. NeurIPS, 2017."
    source:
      paper_report: "workflow/04_paper_digest/round_XX/reports/.../papers/P001.md"
      evidence_claims:
        - "C03"
    allocated_units:
      - "S01.2.2-U001"
    allowed_contexts:
      - "background"
      - "related_work"
      - "gap"
    usage_hint: "用于说明 Transformer 架构对序列建模范式的影响"
```

### 14.4 与其他输出的关系

1. `writing_units.yaml` 中涉及文献引用的 unit，应在 unit 蓝图中添加 `citation_refs: ["vaswani2017attention"]`；
2. `source_allocation.yaml` 的 `paper_sources` 应补充 `citation_tag`；
3. `outline_blueprint.yaml` 应汇总 `citation_plan: "workflow/07_outline/citation_plan.yaml"`；
4. `outline_result.yaml` 应记录 `citation_plan` 输出路径和 citation 数量；
5. `08-section-write` 必须把相关 `citation_specs[]` 传给 writer；
6. `09-assemble` 必须处理所有 `{{cite:tag}}`，未识别 tag 应保留原文并写入 warning。

---

## 15. outline_state.yaml 规则

本 Skill 必须生成：

```text
workflow/07_outline/outline_state.yaml
```

`outline_state.yaml` 是 08-section-write 的循环引擎。

它不只追踪章节，也追踪 writing units。

状态流转：

```text
pending → in_progress → written → approved
```

如果材料不足：

```text
blocked
```

规则：

1. 所有 section（含叶结点和非叶结点）初始为 `pending`，除非缺材料，则为 `blocked`；
2. 所有 writing unit 初始为 `pending`，除非该 unit 缺材料，则为 `blocked`；
3. 非叶结点 section 的状态由其 children（下级 section）的状态聚合；
4. 叶结点 section 的状态由其挂载的 units 的状态聚合；
5. `08-section-write` 应优先写 `priority: high` 且 `status: pending` 的 unit；
6. 每个 unit 必须有 `output_file` 和 `heading_level`；
7. `outline_state.yaml` 的 `section_index` 提供所有 section 的扁平索引，供 08-section-write 快速定位叶结点。

---

## 16. 输出文件结构

本 Skill 生成以下文件：

```text
workflow/07_outline/
  outline_report.md
  volume_budget.yaml
  writing_units.yaml
  source_allocation.yaml
  figure_plan.yaml
  table_plan.yaml
  citation_plan.yaml
  outline_state.yaml
  outline_blueprint.yaml
  context_bundle.yaml          # 供 08 writer agents 使用
  outline_result.yaml
```

文件含义：

| 文件 | 含义 |
|---|---|
| `outline_report.md` | 给人看的申请书逻辑大纲与体量规划说明，含完整大纲标题树（最深 5 级，叶结点标注 🍃） |
| `volume_budget.yaml` | 文档总页数/字数、各章页数/字数（section tree 递归，与 outline_state 同构） |
| `writing_units.yaml` | 每个大纲 section 的 writing units，含 `section_id`、`heading_level`、段落槽位和边界 |
| `source_allocation.yaml` | 每个 writing unit 使用哪些上游证据和材料 |
| `figure_plan.yaml` | 全书图片规划、统一风格、正文插入位置和 Codex 生图提示词 |
| `table_plan.yaml` | 全书表格规划、列结构、行结构、数据来源和 08 自动生成规则 |
| `citation_plan.yaml` | 全书稳定引用 tag、可引用 claim、参考文献条目和 09 编号依据 |
| `outline_state.yaml` | 大纲 section tree（最深 5 级） + units 写作状态追踪 + `section_index` 扁平索引 |
| `outline_blueprint.yaml` | 给 auto 和后续阶段读取的结构化大纲蓝图，含 `section_tree_summary` |
| `outline_result.yaml` | 阶段状态文件 |

写输出文件前，应读取本 Skill `references/` 目录中的对应模板。

模板文件包括：

```text
references/outline_report_template.md
references/volume_budget_template.yaml
references/writing_units_template.yaml
references/source_allocation_template.yaml
references/figure_plan_template.yaml
references/table_plan_template.yaml
references/citation_plan_template.yaml
references/outline_state_template.yaml
references/outline_blueprint_template.yaml
references/outline_result_template.yaml
```

不要使用硬编码绝对路径引用模板。应使用本 Skill 所在目录下的 `references/`。

---

## 17. 执行流程

### 第 1 步：读取模板和输入

1. 读取 L1 主输入；
2. 读取 L2 证据索引；
3. 查找并解析申请书模板；
4. 读取本 Skill references 中的输出模板；
5. 若缺少必要输入，生成 blocked 版 `outline_result.yaml`。

### 第 2 步：建立模板章节骨架

1. 提取模板标题层级；
2. 保留模板官方标题；
3. 判断哪些章节需要补充三级/四级标题；
4. 建立 section tree。

### 第 3 步：映射 helm 方案

将 `scheme_blueprint.yaml` 中的以下内容映射到章节：

- project_positioning；
- selected_problem；
- selected_route；
- system_scheme；
- research_contents；
- validation_plan；
- outline_guidance；
- rejected_problem_directions；
- rejected_routes。

### 第 4 步：映射证据

从 `current_view.md` 和 `evidence_ledger.yaml` 中为章节和 writing units 分配：

- 背景证据；
- 研究现状证据；
- gap 证据；
- baseline；
- 指标；
- 风险；
- 技术启发；
- 可用于申请书正文的 claim。

### 第 5 步：生成体量预算

生成 `volume_budget.yaml`。

必须包括：

- document target；
- section target；
- subsection target；
- expansion density；
- rationale；
- 是否需要用户确认；
- 各 section 目标字数之和必须接近总目标字数。

### 第 6 步：拆解 writing units

生成 `writing_units.yaml`。

要求：

1. **每个 section（不论叶/非叶）至少 1 个 unit**：
   - 非叶结点：1 个 `section_intro` unit（`needs_intro_paragraph` 按 §7.3.1 标准判断）；
   - 叶结点：≥1 个 unit；
2. 重点叶结点（高字数、高论证密度）应有多个 unit；
3. 每个 unit 的 `section_id` 指向所属 section ID；
4. 每个 unit 的 `heading_level` 等于所属 section 的 level；
5. 每个 unit 标注 `is_first_unit_of_section`（决定是否写标题）；
6. 单个 unit 不宜过大（见 §9.2 粒度指南）；
7. 非 `heading-only` 的 section_intro 和所有叶结点 unit 要有 paragraph_slots；
8. 每个 unit 都要有 target_words（`heading-only` section_intro 可为 0）；
9. 每个 unit 都要有 output_file；
10. 每个 unit 要有 avoid 约束。

### 第 7 步：生成 source_allocation

为每个 unit 生成 source allocation。

若单元缺少材料，标记 blocked。

### 第 8 步：生成 figure_plan

生成 `figure_plan.yaml`。

要求：

1. 从 `scheme_blueprint.yaml` 的系统方案、研究内容、技术路线、验证计划中识别必须图；
2. 从模板和 `volume_budget.yaml` 判断每张图的目标位置和篇幅合理性；
3. 为每张图分配 `figure_id`、`target.section_id`、`target.unit_id`、`target.placement_hint`；
4. 为每张图写明 `argument_role`，确保图片服务论证，不作为装饰；
5. 统一 `visual_style`，包括画布比例、配色、字体、布局和禁忌；
6. 生成完整 `codex_prompt_base`，达到可直接复制给 Codex 生图的精细度；
7. 在相关 `writing_units.yaml` unit 中添加 `figure_refs`；
8. 在 `source_allocation.yaml` 的 `figures` 中引用对应 `figure_id`；
9. 若某图需要用户确认素材、前期成果或真实数据，标记 `status: needs_user_confirmation`。

### 第 9 步：生成 table_plan

生成 `table_plan.yaml`。

要求：

1. 从模板、`scheme_blueprint.yaml`、`helm_report.md`、`current_view.md`、`evidence_ledger.yaml` 和 `applicant_profile.md` 中识别适合用表格承载的比较、汇总、计划、指标和基础材料；
2. 为每张表分配 `table_id`、`table_no`、`target.section_id`、`target.unit_id`、`target.placement_hint`；
3. 为每张表写明 `argument_role`，确保表格服务扫描、比较或核对，不替代正文论证；
4. 设计 `columns[]`，每列必须有 `name`、`purpose`、`source`、`required`；
5. 设计 `rows` 生成模式，明确行来自研究内容、模块、验证任务、年度计划、前期成果还是显式行；
6. 设定 `fill_rules`，特别是缺失值、数字来源和是否允许估计性文字；
7. 在相关 `writing_units.yaml` unit 中添加 `table_refs`；
8. 在 `source_allocation.yaml` 的 `tables` 中引用对应 `table_id`；
9. 若核心数据缺失，标记 `status: needs_user_confirmation` 或 `blocked`，不得要求 08 编造数据。

### 第 10 步：生成 citation_plan

生成 `citation_plan.yaml`。

要求：

1. 从 `source_allocation.yaml` 的 `paper_sources` 和 `evidence_claims` 中提取所有可能进入正文的论文；
2. 为每篇论文生成稳定、可读、语义化的 `tag`，不得使用数字编号式 tag；
3. 为每条引用生成 `reference_text`，但不带 `[1]` 这类最终编号；
4. 在相关 `writing_units.yaml` unit 中添加 `citation_refs`；
5. 在 `source_allocation.yaml` 的 `paper_sources` 中补充 `citation_tag`；
6. 标明 `allocated_units`、`allowed_contexts`、`usage_hint`，帮助 writer 在首次介绍工作时正确使用 tag；
7. 若论文元数据不足以生成参考文献条目，标记 `metadata_status: partial`，但仍生成 tag，并在 `outline_result.yaml` 中 warning。

### 第 11 步：生成 outline_state

初始化所有 section 和 unit 状态。

### 第 11.5 步：生成 context_bundle.yaml

**这是 08-section-write 的 writer agents 的共享上下文**。不生成此文件将导致 writer agents 之间出现术语不一致、禁写内容穿透等问题。

1. 读取模板 `references/context_bundle_template.yaml`
2. 从以下来源填充各字段：

| 字段 | 来源 |
|------|------|
| `writing_constraints` | 使用模板默认值（本项目/客观严谨），如有 CLAUDE.md 中的特殊规则则合并 |
| `terminology` | 从 `scheme_blueprint.yaml` 的系统方案和 `current_view.md` 中提取核心术语（每个术语含 term/definition/aliases/forbidden） |
| `argument_chain` | 从 `outline_report.md` 的论证链提取，每个 step 映射到对应 section_id |
| `forbidden_directions` | 从 `decision_log.md` 中提取所有 dropped 和 background_only 方向，含 keyword_triggers |
| `claim_allocations` | 从 `source_allocation.yaml` 中汇总每个 claim 分配到哪些 unit（含 usable_in/forbidden_in） |
| `transition_map` | 从 `outline_state.yaml` 的 depends_on/feeds_into 生成，每个 unit 标注 feeds_into 和 transition_hint |

3. 写入 `workflow/07_outline/context_bundle.yaml`

### 第 12 步：生成 outline_report 和 outline_blueprint

`outline_report.md` 给人看，应包含：

- 项目基本信息；
- 模板来源；
- 大纲说明；
- 逻辑大纲；
- 体量规划摘要；
- writing units 摘要；
- 证据分配摘要；
- 图片规划摘要；
- 表格规划摘要；
- 引用规划摘要；
- blocked 单元；
- 跨节一致性约束。

`outline_blueprint.yaml` 给机器读，应整合：

- template structure；
- volume budget summary；
- unit summary；
- figure plan summary；
- table plan summary；
- citation plan summary；
- helm mapping；
- argument integrity；
- writing workflow hints；
- quality scores。

### 第 13 步：生成 outline_result

写入阶段状态、输出路径、质量评分和下一阶段建议。

### 第 14 步：产出物完整性自检

1. 检查以下文件是否存在且非空：
   - `workflow/07_outline/outline_report.md`
   - `workflow/07_outline/volume_budget.yaml`
   - `workflow/07_outline/writing_units.yaml`
   - `workflow/07_outline/source_allocation.yaml`
   - `workflow/07_outline/figure_plan.yaml`
   - `workflow/07_outline/table_plan.yaml`
   - `workflow/07_outline/citation_plan.yaml`
   - `workflow/07_outline/outline_state.yaml`
   - `workflow/07_outline/outline_blueprint.yaml`
   - `workflow/07_outline/context_bundle.yaml`
   - `workflow/07_outline/outline_result.yaml`
2. 将验证结果写入 `outline_result.yaml` 的 `integrity` 字段：

```yaml
integrity:
  all_outputs_present: true/false
  checked_at: "<当前时间>"
  missing_outputs: []
  warnings: []
```

3. 若 `all_outputs_present: false` → 不声称阶段完成，阻塞 08-section-write 的启动。

---

## 18. 阻塞规则

| 情况 | 处理方式 |
|---|---|
| `scheme_blueprint.yaml` 缺失 | 阻塞，提示先完成 `06-helm` |
| `helm_report.md` 缺失 | 阻塞，提示先完成 `06-helm` |
| `topic.md` 缺失 | 阻塞，提示先完成 `01-topic` |
| 模板缺失 | 不阻塞，使用内置通用提纲 |
| pandoc 无法解析模板 | 不阻塞，使用内置通用提纲 |
| 某必写章节缺材料 | 整体不阻塞，该章节或 unit 标记 blocked |
| 目标页数/字数不明确 | 不阻塞，使用默认值，并标记 `needs_user_confirmation: true` |
| 核心研究内容无法拆成 writing units | 阻塞或标记核心 unit blocked，提示需要回到 helm |

---

## 19. 最终响应格式

执行成功后，向用户简要报告：

```text
已完成第 07 阶段：申请书内容架构与体量规划。

模板来源：[模板文件名 / 内置提纲]
目标体量：[X 页 / X 字，是否需要确认]
大纲标题树：[L1 X 个，L2 X 个，L3 X 个，L4 X 个，L5 X 个]（最大深度 X 级）
叶结点数：X 个
生成 writing units：[X 个，其中 high priority X 个]
blocked units：[X 个]

输出文件：
- workflow/07_outline/outline_report.md（含完整大纲标题树 + 叶结点标注）
- workflow/07_outline/volume_budget.yaml（section tree 递归体量预算）
- workflow/07_outline/writing_units.yaml（叶结点→units，含 heading_level）
- workflow/07_outline/source_allocation.yaml
- workflow/07_outline/figure_plan.yaml（图片规划 + Codex 生图提示词）
- workflow/07_outline/table_plan.yaml（表格规划 + 08 自动生成规则）
- workflow/07_outline/citation_plan.yaml（稳定引用 tag + 参考文献条目）
- workflow/07_outline/outline_state.yaml（section tree 最深 5 级 + section_index）
- workflow/07_outline/outline_blueprint.yaml
- workflow/07_outline/outline_result.yaml

下一步建议：
进入 08-section-write，从 writing_units.yaml 中 priority=high 且 status=pending 的第一个 unit 开始写作。
```

如果阻塞：

```text
无法完成 07-outline，因为缺少必要输入：
- ...

请先完成：
- ...
```

---

## 20. 质量要求

1. 所有正文输出使用中文；
2. 可保留必要英文术语；
3. 不硬编码项目绝对路径；
4. 不读取、修改、创建 `./workflow/proposal_state.yaml`；
5. 不写申请书正文；
6. 不重新做 helm；
7. 不重新做 synthesis；
8. 不编造论文、数据、结论或团队成果；
9. 模板中的官方标题必须保留；
10. 大纲必须生成完整标题树（最深 5 级），叶结点显式标注；
11. 必须生成体量预算（section tree 递归，与 outline_state 同构）；
12. 必须生成 writing units，每个 unit 的 `section_id` 指向所属 section ID；
13. 每个 unit 必须有 `heading_level`（等于所属 section 的 level）；
14. 必须生成 source allocation；
15. 必须生成 figure_plan，每张必需图片必须有可直接用于 Codex 生图的 `codex_prompt_base`；
16. 必须生成 table_plan，每张必需表格必须有列结构、行结构、数据来源和 08 自动生成规则；
17. 必须生成 citation_plan，每条可进入正文的论文来源必须有稳定 tag 和参考文献条目；
18. 必须初始化 outline_state（section tree + section_index + unit states）；
19. 每个重点 writing unit 必须有 target_words 和 paragraph_slots；
20. 每个涉及文献、gap、baseline 或创新性的 unit 必须有证据来源；
21. dropped 方向不得进入主线叶结点；
22. background_only 方向只能作为背景叶结点简要出现；
23. 输出必须服务 08-section-write；
24. 最终响应中不要执行其他 Skill。

---

## 附录A：通用申请书结构与写作规范

以下内容为生成大纲时的写作风格参考。具体章节标题以用户模板为准。

### A.1 标题不可随意修改

如果模板提供了固定标题，必须严格保留：

- 不修改一级、二级标题；
- 不遗漏必写标题；
- 不在规定提纲之外自创一级或二级标题；
- 可在规定标题下添加三级及更深标题。

### A.2 内置通用提纲

当 `references/` 下无模板文件时，使用以下通用提纲作为骨架：

```text
（一）立项依据
  1. 研究意义
  2. 国内外研究现状
  3. 现有方法不足与本项目切入点

（二）研究内容
  1. 研究目标
  2. 研究内容
  3. 关键科学/技术问题
  4. 研究方案与技术路线
  5. 特色与创新
  6. 年度研究计划

（三）研究基础
  1. 研究基础与可行性分析
  2. 工作条件
  3. 相关科研项目情况
  4. 已完成项目情况

（四）其他需要说明的情况
```

### A.3 内容展开原则

长文档不是靠空话扩写，而是靠写作单元展开。

每个 writing unit 应尽量包含：

1. 明确论点；
2. 背景或上下文；
3. 技术解释；
4. 与本项目方案的关系；
5. 证据或依据；
6. 指标、约束或边界；
7. 验证方式；
8. 与前后文的衔接。

### A.4 章节逻辑链

```text
立项依据（Why）
  → 研究目标（What goal）
  → 研究内容（What to do）
  → 关键问题（Key problems）
  → 研究方案/技术路线（How）
  → 创新点（Novelty）
  → 验证计划（How to prove）
  → 研究基础（Why us）
```

### A.5 文风要求

推荐：

- 客观严谨；
- 自信但不夸大；
- 多用短句和动宾结构；
- 术语一致；
- 论证链清楚。

严格避免：

- “填补空白”；
- “国内领先”；
- “国际先进”；
- “首次提出”；
- 无依据的量化数据；
- 泛泛而谈的意义描述。
