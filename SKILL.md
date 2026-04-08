---
name: prd-to-system-analysis
description: Generates system analysis documents from PRD and project context using an SDD state-machine workflow. Persists intermediate artifacts, supports resume/manual-edit sync, supports rollback on requirement corrections, and requires explicit DOC final gate before template drafting. Use when users mention PRD, 系分文档, 系统分析, 方案设计, PlantUML, or template-driven design docs.
---

# PRD to 系分（SDD 状态机版）

**Version:** 2.2 · **Logical id:** `prd_to_system_analysis`

## Input

- `prd` (conditional): PRD 正文或 PRD 文件路径
- `project_positioning`（可选）：工程定位、边界、上下游系统说明
- `template_path`（可选）：系分模板路径，默认 `templates/系分模版.md`
- `project_root`（可选）：工程根目录，默认当前工作目录
- `output_path`（可选）：最终系分输出路径；多文档时可作为输出目录
- `historical_docs_dir`（可选）：历史 PRD/系分样本目录
- `artifact_dir`（可选）：中间产物目录（优先级最高）
- `lang`（可选）：`zh-CN` | `en`（默认按用户本轮语言）
- `start_phase`（可选）：`AUTO` | `SPEC` | `SPLIT` | `CURRENT_STATE` | `PLAN` | `DOC` | `REVIEW`（默认 `AUTO`）
- `manual_edit_mode`（可选）：`on` | `off`（默认 `on`，接管用户手改中间文档）
- `export_format`（可选）：`none` | `docx` | `doc` | `both`（默认 `none`，用于交付阶段导出）

## Output

- 分阶段中间文档：`spec/split/current-state/plan/review`
- 最终系分文档（按模板，支持多系统多文档）
- 可恢复状态文件：`analysis.state.<lang>.json`
- 可选交付物：`system-analysis.<lang>.docx/.doc`（仅在用户明确要求时导出）

## Language mode

1. `lang=zh-CN`：全量中文输出。
2. `lang=en`：全量英文输出。
3. 未指定 `lang`：按用户本轮语言自动识别。
4. 每轮输出前先读取：
   - `zh-CN` -> [templates/zh-CN.md](templates/zh-CN.md)
   - `en` -> [templates/en.md](templates/en.md)

## PRD 输入解析（文本/文件）

1. `prd` 像路径且文件存在时按文件读取，否则按内联文本处理。
2. 优先支持：`.md`、`.txt`、可读文本文件。
3. 对 `.doc` / `.docx`：若无法可靠解析，不得臆造内容，要求用户转 `.md`/纯文本或直接粘贴。
4. 若 `prd` 未显式提供：
   - 自动在 `project_root` 搜索 PRD 候选（如 `*PRD*.md`、`【PRD】*.md`）；
   - 仅 1 个候选时自动采用；
   - 多个候选时要求用户选择；
   - 无候选时要求用户提供 PRD。
5. 进入首个阶段前必须有可解析 PRD；若信息不足，先提问补齐关键缺口。

## Artifact directory（中间产物落盘）

`artifact_root` 按以下优先级解析：

1. `artifact_dir`（用户显式指定）
2. 环境变量 `SDD_ARTIFACT_DIR`
3. `project_root`

建议在全局目录下使用子目录 `${SDD_ARTIFACT_DIR}/${project_slug}/`，避免多项目冲突。

### Mandatory artifacts（必须落盘）

在 `artifact_root` 下维护（按语言区分）：

- `spec.<lang>.md`
- `split.<lang>.md`
- `current-state.<lang>.md`
- `plan.<lang>.md`
- `review.<lang>.md`
- `analysis.state.<lang>.json`

建议最终正文命名：
- 单文档：`system-analysis.<lang>.md`
- 多文档：`system-analysis.<system>.<lang>.md`

## Resume behavior（恢复会话）

每次进入 `INIT` 时先执行恢复检查：

1. 定位 `artifact_root` 与 `lang`。
2. 若存在 `analysis.state.<lang>.json` 或阶段文档，汇总最近进度。
3. 询问用户：`resume`（继续历史）或 `restart`（从头开始）。
4. `resume`：从最近未完成阶段继续，优先复用已落盘中间文档。
5. `restart`：保留旧文档（可时间戳备份）并重启流程。

## Start-from-any-phase（任意阶段启动）

起始阶段解析优先级：

1. `start_phase`（用户显式指定）
2. `analysis.state.<lang>.json` 历史状态（resume）
3. 默认 `SPEC`

入口校验规则：

- `SPEC`：直接进入。
- `SPLIT`：优先读取 `spec.<lang>.md`；缺失则补最小需求基线或回退 `SPEC`。
- `CURRENT_STATE`：优先读取 `split.<lang>.md`；缺失则补切片映射或回退 `SPLIT`。
- `PLAN`：优先读取 `current-state.<lang>.md`；缺失则补现状证据或回退 `CURRENT_STATE`。
- `DOC`：必须已确认 `PLAN`，且通过“最终成文确认门禁”；否则禁止进入。
- `REVIEW`：必须存在最终系分文档；否则先完成 `DOC`。

若上游产物缺失，不得假装已完成。必须显式告知缺口，并让用户选择：

1. 补齐缺失产物后继续
2. 回退到上一阶段生成
3. 以用户提供摘要作为临时上游基线

## Manual edit synchronization（人工修改接管）

当 `manual_edit_mode=on`：

1. 每次阶段开始前重新读取已有中间文档。
2. 若检测到文件与状态记录不一致（时间戳/校验和变化），视为用户手改。
3. 提示用户选择：
   - `adopt`：采用手改版本继续
   - `merge`：与当前草稿合并
   - `regenerate`：基于最新输入重生成该阶段
4. 未确认策略前，不覆盖用户手改内容。
5. 若用户仅回复“ok/继续/确认”且未指定策略，默认按 `adopt` 处理，并在状态文件记录该默认行为。

## Intent normalization（简短确认语义归一）

当用户仅回复简短确认词（如 `ok`/`确认`/`继续`）时，按当前等待上下文解析：

1. 等待阶段确认块 -> 视为本阶段确认通过。
2. 等待 DOC 最终门禁 -> 视为“确认生成”。
3. 等待手改策略 -> 默认 `adopt`。
4. 等待方案选择 -> 不自动推断，必须二次确认具体选项（A/B/...）。

## Requirement correction rollback（需求纠偏回退协议）

当用户出现“理解错误/回溯/改需求/方案重做/范围变更”等信号：

1. 识别受影响最早阶段（通常是 `SPEC` 或 `SPLIT`）。
2. 回退状态到该阶段 `*_CONFIRMING`。
3. 将下游产物标记为 `stale_due_to_spec_change`（或等价状态）。
4. 保留旧文档，不删除；按新基线重生成并重新确认。
5. 在状态文件记录：
   - `rollback.from_phase`
   - `rollback.trigger`
   - `rollback.reason`
   - `needs_regeneration`

## Data evolution & compatibility protocol（字段/类型演进必检）

若 PRD 包含“新增字段/新增类型/口径变化/旧逻辑兼容”：

1. `CURRENT_STATE` 必须评估历史逻辑影响。
2. `PLAN` 必须给出兼容方案，至少包含：
   - 新字段写入逻辑
   - 历史数据回填策略
   - 旧作业/旧查询过滤策略
   - 灰度与回滚路径
3. `DOC` 必须落“历史逻辑影响与兼容处理”段。
4. `REVIEW` 必须校验“字段演进 -> 兼容策略 -> 回归用例”闭环。

## Naming governance（命名约束同步）

若用户在 `PLAN` 阶段提出命名规范（如“统一使用债转凭证命名”）：

1. 必须更新 `plan.<lang>.md` 中类/方法/接口命名。
2. 必须同步到 `DOC` 正文，避免“方案命名”与“正文命名”不一致。
3. `REVIEW` 阶段需增加命名一致性检查。
4. 对“仍在使用的常规流程/常规类型”，禁止使用 `LEGACY`/`历史` 语义命名；应使用中性业务维度命名（如 `TERM_ASSET_MGMT`）。

## End-to-end flow

```text
[START: AUTO|SPEC|SPLIT|CURRENT_STATE|PLAN|DOC|REVIEW]
-> 入口校验
-> 手改同步
-> SPEC -> SPLIT -> CURRENT_STATE -> PLAN
-> DOC_FINAL_GATE（必须用户确认）
-> DOC（模板成文）
-> REVIEW
-> (可选) EXPORT（doc/docx）
-> DONE
```

## Non-negotiable rules

1. 默认顺序执行，可显式跳转，但必须通过入口校验。
2. Unknowns 必须显式标记 `❓`，禁止静默假设。
3. 每阶段完成必须落盘并更新 `analysis.state.<lang>.json`。
4. `PLAN` 阶段必须给至少两案（A/B）并等待用户选择。
5. 用户未明确选项前，不得进入 `DOC`。
6. 未经最终确认，不得进入 `DOC` 按模板生成正文。
7. 必须先读 PRD，再读工程现状，再做方案，顺序不可反。
8. 大 PRD 必须先做切片映射并确认，再进入方案与成文。
9. 不得编造接口、表结构、现状能力；必须附证据路径/模块。
10. 图统一使用 `PlantUML`，不得使用 Mermaid。
11. 需求纠偏时必须回退并标记下游产物失效，不得“带病继续”。
12. 涉及字段/类型演进时必须给兼容方案与迁移策略。
13. 命名规范一旦被用户确认，后续阶段必须一致。
14. 历史样本仅用于风格与边界对齐，冲突时以当前 PRD + 现状证据为准。

## State machine

| State | Meaning |
|------|---------|
| `INIT` | 读取输入并检查是否可恢复 |
| `ENTRY_VALIDATING` | 校验阶段跳转前置条件 |
| `ARTIFACT_SYNCING` | 同步用户手改中间文档 |
| `ROLLBACK_SYNCING` | 需求纠偏后的回退与失效标记 |
| `SPEC_DRAFT` / `SPEC_CONFIRMING` | 需求分析与确认 |
| `SPLIT_DRAFT` / `SPLIT_CONFIRMING` | 切片映射与确认 |
| `CURRENT_STATE_DRAFT` / `CURRENT_STATE_CONFIRMING` | 现状能力盘点与确认 |
| `PLAN_DRAFT` / `PLAN_CONFIRMING` | A/B 方案与用户选择 |
| `DOC_PENDING_FINAL_CONFIRM` | 等待最终成文确认（强门禁） |
| `DOC_DRAFT` / `DOC_CONFIRMING` | 按模板生成系分正文并确认 |
| `REVIEW_DRAFT` / `REVIEW_CONFIRMING` | 一致性校验与交付确认 |
| `EXPORT_DRAFT` / `EXPORT_CONFIRMING` | 可选导出 doc/docx |
| `DONE` | 结束 |

Forbidden：未通过门禁直接生成正文；检测到手改后直接覆盖写盘。

## Mandatory output blocks（每轮必带）

按 `lang` 读取模板并使用：

- Per-turn Footer / 每轮三件套
- Phase Confirmation Block / 阶段确认块
- Final Gate Before DOC / 最终成文门禁块
- Resume Prompt Block / 历史恢复提示块
- Phase-jump Entry Validation Block / 阶段跳转入口校验提示块
- Manual Edit Detected Block / 检测到人工修改提示块
- Rollback Prompt Block / 需求纠偏回退提示块

## Phase routing（阶段路由）

### Phase: SPEC

1. 读取 [spec.md](spec.md)。
2. 读取 [templates/spec.md](templates/spec.md) 对应语言骨架。
3. 产出并写入 `spec.<lang>.md`，进入确认。

### Phase: SPLIT

1. 读取 [split.md](split.md)。
2. 读取 [templates/split.md](templates/split.md) 对应语言骨架。
3. 产出并写入 `split.<lang>.md`，进入确认。

### Phase: CURRENT_STATE

1. 读取 [current-state.md](current-state.md)。
2. 读取 [templates/current-state.md](templates/current-state.md) 对应语言骨架。
3. 产出并写入 `current-state.<lang>.md`，进入确认。
4. 若存在字段/类型演进，必须增加历史逻辑影响盘点。

### Phase: PLAN

1. 读取 [plan.md](plan.md)。
2. 读取 [templates/plan.md](templates/plan.md) 对应语言骨架。
3. 强制输出 A/B 方案并要求用户选择。
4. 若有字段/类型演进，必须给兼容与迁移方案。
5. 若用户要求命名规范，必须写入方案并后续保持一致。
6. 写入 `plan.<lang>.md`，进入确认。

### Phase: DOC

1. 读取 [doc.md](doc.md)。
2. 读取 [templates/doc.md](templates/doc.md) 对应语言骨架。
3. 再读取最终模板 [templates/系分模版.md](templates/系分模版.md)。
4. 只有用户通过“Final Gate Before DOC”后，才可生成最终系分正文。

### Phase: REVIEW

1. 读取 [review.md](review.md)。
2. 读取 [templates/review.md](templates/review.md) 对应语言骨架。
3. 产出并写入 `review.<lang>.md`，进入交付确认。

### Phase: EXPORT（可选）

1. 仅当用户明确要求导出 `doc/docx` 时执行。
2. 基于最终 `system-analysis.<lang>.md` 导出，不改动正文语义。
3. 导出后记录输出文件路径到状态文件。

## Domain heuristics（领域启发式）

### 历史样本对齐

若存在历史 PRD 与系分文档，先对齐风格、抽象粒度、系统切分方式：

- `【PRD】微粒贷.md` -> `【lcs】微粒贷二期.md`、`【instloancore】微粒贷代偿后还款系分.md`
- `【PRD】费率计算器升级.md` -> `【lcs】【系分】费率计算器升级.md`

历史样本仅用于风格对齐，冲突时以“当前 PRD + 工程现状证据”为准。

### 微粒贷类（多系统）

- 默认按“期数 x 还款场景 x 系统边界”拆分。
- 若涉及 `lcs` 与 `instloancore`，默认输出多份系分文档。
- 单列“资方口径 vs 我方口径”一致性处理。

### 费率计算器类（规则密集）

- 必须输出规则矩阵：`Xcap`、未来预估应收、未来预估抵扣、预估减免金额。
- 必须覆盖跳期/顺序等校验规则、兼容字段、灰度开关、回归范围。

## 触发关键词

- `PRD`
- `系分文档` / `系统分析`
- `按模板输出`
- `方案设计` / `A B 方案`
- `读取工程现状`
- `PlantUML` / `组件图` / `时序图` / `ER图`
- `回溯` / `理解错误` / `需求纠偏`
- `字段新增` / `类型新增` / `兼容处理`
- `doc` / `docx` / `导出文档`

## Additional resources

- [spec.md](spec.md) — SPEC 阶段规则
- [split.md](split.md) — SPLIT 阶段规则
- [current-state.md](current-state.md) — CURRENT_STATE 阶段规则
- [plan.md](plan.md) — PLAN 阶段规则
- [doc.md](doc.md) — DOC 阶段规则
- [review.md](review.md) — REVIEW 阶段规则
- [templates/zh-CN.md](templates/zh-CN.md) — 中文模板块（footer/confirm/gate/resume）
- [templates/en.md](templates/en.md) — 英文模板块（footer/confirm/gate/resume）
- [templates/spec.md](templates/spec.md) — SPEC 固定骨架
- [templates/split.md](templates/split.md) — SPLIT 固定骨架
- [templates/current-state.md](templates/current-state.md) — CURRENT_STATE 固定骨架
- [templates/plan.md](templates/plan.md) — PLAN 固定骨架
- [templates/doc.md](templates/doc.md) — DOC 阶段固定骨架
- [templates/review.md](templates/review.md) — REVIEW 阶段固定骨架
- [templates/系分模版.md](templates/系分模版.md) — 默认系分模板
- [examples.md](examples.md) — 历史样本“输入 PRD -> 输出骨架”示例
