---
name: prd-to-design
description: >-
  Turn a PRD into 系分文档 / system design docs (with Mermaid) via the SDD flow
  CONTEXT_SCAN -> SPEC -> SPLIT -> IMPACT_SCAN -> PLAN -> DOC -> REVIEW. Use when
  a PRD needs 系分、系统分析、方案设计、架构设计、概要/详细/接口设计、按模板成文、
  A/B 方案、多系统/上下游影响、评审流程、Mermaid 图、字段/类型新增、历史兼容、
  工程画像/现状、仓库摸底. Not for direct PRD-to-code, or when a finished design
  doc should drive implementation; if ambiguous, ask: 系分 / 直接代码 / 已有系分写代码.
---

# PRD to 系分（SDD 状态机版）

**Version:** 2.4 · **Logical id:** `prd_to_design`

## Input

- `prd` (conditional): PRD 正文或 PRD 文件路径
- `project_positioning`（可选）：工程定位、边界、上下游系统说明
- `template_path`（可选）：系分模板路径，默认 `templates/系分模版.md`
- `project_root`（可选）：工程根目录，默认当前工作目录
- `output_path`（可选）：最终系分输出路径；多文档时可作为输出目录
- `historical_docs_dir`（可选）：历史 PRD/系分样本目录
- `artifact_dir`（可选）：中间产物基础目录（显式指定，优先级最高；缺省时按"PRD 文件目录 → 工作目录"回退，并自动追加 `prd-to-design/` skill 子目录避免与其它 SDD skill 冲突，详见 Artifact directory 章节）
- `session_id`（可选）：会话唯一 ID；未指定时优先使用运行时会话 ID
- `lang`（可选）：`zh-CN` | `en`（默认按用户本轮语言）
- `start_phase`（可选）：`AUTO` | `CONTEXT_SCAN` | `SPEC` | `SPLIT` | `IMPACT_SCAN` | `PLAN` | `DOC` | `REVIEW`（默认 `AUTO`）
- `manual_edit_mode`（可选）：`on` | `off`（默认 `on`，接管用户手改中间文档）
- `export_format`（可选）：`none` | `docx` | `doc` | `both`（默认 `none`，用于交付阶段导出）

## Output

- 分阶段中间文档：`context/spec/split/impact/plan/review`
- 最终系分文档（按模板，支持多系统多文档）
- 可恢复状态文件：`prd-to-design.state.<lang>.json`
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

中间产物默认与 PRD/工程贴近落盘，且强制隔离在以**当前 skill 名命名**的子目录下，避免与并列的 `prd-to-code` / `design-to-code` skill 互相覆盖（这三个 skill 共享相似的中间文件名）。

落盘路径分两步解析：先确定 `parent_dir`（上层目录），再按规则得到 `base_dir`，最后拼接 `session_id`。

**Skill segment（本 skill 固定值）：** `prd-to-design`

### Step 1: `parent_dir` 解析优先级（高 → 低）

1. **PRD 文件所在目录**：当 `prd` 解析为文件路径（用户显式提供、`@文件` 引用、或自动在 `project_root` 搜索得到的 PRD 文件）时，使用该 PRD 文件的**父目录**
2. **工作目录**：当 PRD 为内联文本、无可定位的源文件时，使用 `project_root`（未提供则为当前工作目录 `cwd`）

禁止回退到 skill 自身目录（`SKILL.md` 所在目录），避免在 skill 仓库内堆积业务产物。

### Step 2: `base_dir` 解析

1. **显式指定（最高优先级）** — 直接整体替换默认 `base_dir`，**不**自动追加 skill 名（视用户已自行决定路径布局）
   - 输入参数 `artifact_dir`（用户在调用时显式指定）
   - 环境变量 `SDD_ARTIFACT_DIR`（全局显式配置）
   - 二者同时存在时，`artifact_dir` 优先于环境变量
   - 若用户显式指定的目录与其它并列 skill 的默认目录相同，必须给出冲突告警并要求用户确认
2. **默认（缺省）**：`base_dir = <parent_dir>/prd-to-design`，标记 `base_dir_source=default`
   - ⚠️ 该路径由系统推导得出，**非用户显式指定**：必须先通过 Step 4「默认目录确认门禁」，由用户确认沿用或改路径后，才能创建目录与落盘
   - 若该路径不存在，需在用户确认后再自动创建
   - 若该路径已存在但属于另一 skill（含 `design-to-code.state.<lang>.json` / `prd-to-code.state.<lang>.json` 等他 skill 独占产物），必须告警并要求用户决策

### Step 3: `session_id` 与最终目录

`session_id` 按以下顺序解析：

1. 运行时提供的会话唯一 ID
2. 输入参数 `session_id`
3. 若仍不可得，要求用户补充 `session_id` 后再继续（不得回退到固定目录）

最终产物目录：

- `artifact_root = <base_dir>/<session_id>`

### Step 4: 默认目录确认门禁（仅当 `base_dir_source=default`）

当 `base_dir` 走缺省默认（用户未通过 `artifact_dir` / `SDD_ARTIFACT_DIR` 显式指定）时，进入 `ARTIFACT_DIR_CONFIRMING` 状态，在落盘任何中间产物前必须先确认：

1. 用「默认产物目录确认块」回显 `parent_dir` / `base_dir` / `artifact_root`，请用户确认或改路径。
2. 用户响应：
   - 「确认」/「OK」/「继续」-> 沿用默认 `artifact_root`，置 `artifact_dir_confirmed=true` 后进入后续阶段。
   - 直接给出新路径 -> 将其按显式指定处理（整体替换 `base_dir`，**不**追加 skill 名），重算 `artifact_root` 并回显一次，再置 `artifact_dir_confirmed=true`。
3. **确认前不得创建目录、不得写入任何中间产物**（含 `prd-to-design.state.<lang>.json` 自身）。
4. 当 `base_dir_source` 为 `artifact_dir` / `SDD_ARTIFACT_DIR`（显式指定）时，跳过本门禁，视为用户已决定路径，仅在 Echo & audit 回显。
5. 确认结果写入 `prd-to-design.state.<lang>.json`（`artifact_dir_confirmed=true` 与最终 `artifact_root`）；Resume 时若已确认过则不再重复询问。

### Echo & audit（每轮 INIT 必须回显）

每轮进入 `INIT` 时必须把以下字段显式回显给用户，便于人工核对：

- `parent_dir`、`parent_dir_source`（来源：`prd_file_dir` / `project_root` / `cwd`）
- `base_dir`、`base_dir_source`（来源：`artifact_dir` / `SDD_ARTIFACT_DIR` / `default(<parent_dir>/prd-to-design)`）
- `artifact_root`
- `artifact_dir_confirmed`（默认目录是否已确认；`base_dir_source=default` 且未确认时，回显后必须立即进入 Step 4 确认门禁）

### Mandatory artifacts（必须落盘）

在 `artifact_root` 下维护（按语言区分）：

- `context.<lang>.md`（CONTEXT_SCAN 工程画像，无 PRD 依赖）
- `spec.<lang>.md`
- `split.<lang>.md`
- `impact.<lang>.md`（PRD 范围内深扫与历史影响评估）
- `plan.<lang>.md`
- `review.<lang>.md`
- `prd-to-design.state.<lang>.json`

建议最终正文命名：
- 单文档：`system-analysis.<lang>.md`
- 多文档：`system-analysis.<system>.<lang>.md`

## State file schema（状态文件结构）

`prd-to-design.state.<lang>.json` 是恢复、手改检测、回退的唯一事实源，至少包含以下结构：

```json
{
  "current_state": "SPEC_CONFIRMING",
  "last_updated": "<iso8601>",
  "lang": "zh-CN",
  "artifact_root": "<base_dir>/<session_id>",
  "artifact_dir_confirmed": true,
  "selected_option": null,
  "artifacts": {
    "context": { "path": "context.zh-CN.md", "content_sha256": "<sha256>", "written_at": "<iso8601>", "source": "skill", "stale": false },
    "spec":    { "path": "spec.zh-CN.md",    "content_sha256": "<sha256>", "written_at": "<iso8601>", "source": "skill", "stale": false }
  },
  "final_docs": [],
  "rollback": null
}
```

字段约定：

- `artifacts.<phase>` 必须是结构化对象，**不得**退化为布尔或纯路径字符串（否则手改检测失去基线）。
- `content_sha256`：对**实际写盘的文件内容**（UTF-8 字节）计算的 SHA-256，是手改检测的**唯一判定基线**；每次由 skill 写盘后必须立即回算并回写。
- `written_at`：本次 skill 写盘时间（ISO-8601），仅作展示/排序，**不**作为手改判定依据。
- `source`：`skill`（最近一次内容由本 skill 写入）| `manual`（已确认采用用户手改版本，`adopt` 后置为该值）。
- `stale`：需求纠偏回退时置 `true`（等价 `stale_due_to_spec_change`），表示该产物待重生成。
- `final_docs`：最终正文数组，每项结构同 `artifacts.<phase>`（含 `path` / `content_sha256` / `written_at` / `source`）。

## Skill self-update check（启动版本自检）

本 skill 以 git 仓库形式分发（通常经软链接挂载到 `~/.cursor/skills/` 或 `~/.claude/skills/`）。每个会话**仅在首次进入 `INIT` 时执行一次** `UPDATE_CHECKING`：只读地检测远程分支是否领先本地，若本地落后则提示用户手动拉取最新版本。**只检测、不自动拉取、不改动任何文件。**

1. 定位 `skill_dir`：本 `SKILL.md` 所在目录（若为软链接，`git` 会自动解析到真实仓库）。
2. 静默执行下列命令，**任一步失败即跳过自检**（不报错、不阻塞主流程、不打断用户）：

   ```bash
   git -C "<skill_dir>" rev-parse --is-inside-work-tree         # 非 git 仓库 -> 跳过
   git -C "<skill_dir>" fetch --quiet                           # 无网络 / 无 remote -> 跳过
   git -C "<skill_dir>" rev-list --left-right --count @...@{u}  # 输出两列：ahead behind
   ```

3. 判定与提示：
   - `behind > 0`（本地落后远程）→ 输出「Skill 更新提示」，告知落后提交数并给出手动更新命令，是否更新由用户决定：
     - `zh-CN`：检测到 `prd-to-design` 有新版本（本地落后远程 {behind} 个提交）。建议手动更新：`git -C "<skill_dir>" pull --ff-only`；更新后请重新触发本 skill 生效，本次会话仍使用当前版本。
     - `en`：A newer version of `prd-to-design` is available ({behind} commit(s) behind). Update manually: `git -C "<skill_dir>" pull --ff-only`, then re-trigger the skill. The current session keeps the existing version.
   - `behind = 0`（已是最新）→ 不输出更新提示，不打扰用户。
   - 无 upstream / detached HEAD / fetch 失败 / 非 git 仓库 → 静默跳过。
4. **绝不自动执行 `git pull`**：避免会话进行中变更 skill 文件导致行为不一致。
5. 自检结果可写入状态文件（如 `prd-to-design.state.<lang>.json` 的 `update_check.last_checked_at` / `update_check.behind`），**同一会话不重复 fetch**。

## Resume behavior（恢复会话）

每次进入 `INIT` 时先执行恢复检查：

1. 基于 `base_dir + session_id` 定位 `artifact_root` 与 `lang`。
2. 若存在 `prd-to-design.state.<lang>.json` 或阶段文档，汇总最近进度。
3. 询问用户：`resume`（继续历史）或 `restart`（从头开始）。
4. `resume`：从最近未完成阶段继续，优先复用已落盘中间文档；复用前必须先进入 `ARTIFACT_SYNCING`，对所有已落盘产物按 `content_sha256` 做手改检测（见 Manual edit synchronization），命中差异先按 `adopt`/`merge`/`regenerate` 处理再继续。
5. `restart`：保留旧文档（可时间戳备份）并重启流程。
6. 若历史状态已含 `artifact_dir_confirmed=true`，沿用既有 `artifact_root`，不再触发 Step 4 默认目录确认门禁。

## Start-from-any-phase（任意阶段启动）

起始阶段解析优先级：

1. `start_phase`（用户显式指定）
2. `prd-to-design.state.<lang>.json` 历史状态（resume）
3. 默认 `CONTEXT_SCAN`（首次会话）；存在已确认的 `context.<lang>.md` 时回落到 `SPEC`

入口校验规则：

- `CONTEXT_SCAN`：无前置要求，直接进入；仅做仓库现状轻量盘点，不读 PRD。
- `SPEC`：优先读取 `context.<lang>.md`（无则在 SPEC 内嵌轻量补扫并提示用户后续仍可独立完成 CONTEXT_SCAN）；同时需有可解析 PRD。
- `SPLIT`：优先读取 `spec.<lang>.md`；缺失则补最小需求基线或回退 `SPEC`。
- `IMPACT_SCAN`：优先读取 `split.<lang>.md`；缺失则补切片映射或回退 `SPLIT`。
- `PLAN`：优先读取 `impact.<lang>.md`；缺失则补现状证据或回退 `IMPACT_SCAN`。
- `DOC`：必须已确认 `PLAN`，且通过“最终成文确认门禁”；否则禁止进入。
- `REVIEW`：必须存在最终系分文档；否则先完成 `DOC`。

若上游产物缺失，不得假装已完成。必须显式告知缺口，并让用户选择：

1. 补齐缺失产物后继续
2. 回退到上一阶段生成
3. 以用户提供摘要作为临时上游基线

## Manual edit synchronization（人工修改接管）

当 `manual_edit_mode=on`，在 `INIT`（含 resume）之后、且每个阶段 `*_DRAFT` 之前，必须先进入 `ARTIFACT_SYNCING` 做一次手改检测：

1. 对 `artifacts`（及 `final_docs`）中**所有已落盘产物**（不只当前阶段）逐个重新读取磁盘内容并计算 `SHA-256`。
2. 与状态文件中持久化的 `content_sha256`（baseline）逐一比对：
   - **判定基线唯一为 `content_sha256`**：当前内容指纹 ≠ baseline ⇒ 视为用户手改。
   - 文件时间戳（mtime）仅可作辅助提示，**不得**作为判定依据——skill 自身写盘也会刷新 mtime，无法区分作者。
   - 若某产物在 state 中缺少 `content_sha256`（历史遗留状态文件），不得静默放过：提示"无法确认是否被手改"，并要求补一次指纹基线后再继续。
3. 命中手改时，对每个变更产物提示用户选择：
   - `adopt`：采用手改版本继续，将该产物 `source` 置为 `manual`，并把 `content_sha256` 重算为当前值
   - `merge`：与当前草稿合并，合并结果重新写盘后刷新 `content_sha256`，`source=skill`
   - `regenerate`：基于最新输入重生成该阶段，写盘后刷新 `content_sha256`，`source=skill`
4. 未确认策略前，不覆盖用户手改内容。
5. 若用户仅回复“ok/继续/确认”且未指定策略，默认按 `adopt` 处理，并在状态文件记录该默认行为。
6. 若手改的是上游产物（如改了 `spec` 但当前在 `PLAN`），按"需求纠偏回退协议"评估是否需要把下游产物 `stale` 置位并重生成。

## User confirmation interaction（AskUserQuestion 交互确认 · 强制）

所有「需要等待用户确认/选择」的门禁点，必须通过 `AskUserQuestion` 工具（或当前运行环境的等价结构化提问工具，如 Cursor 的 `AskQuestion`）发起交互式提问收集决策，**不得**只把确认块写进文档/普通回复文本里被动等待用户自行打字。

1. 适用范围（本 skill 所有等待用户输入的状态与场景）：
   - `ARTIFACT_DIR_CONFIRMING`：选项 `confirm`（沿用默认目录，推荐）/ 用户经 Other 给新路径
   - Resume 检查：选项 `resume` / `restart`
   - 各阶段 `*_CONFIRMING`（CONTEXT_SCAN / SPEC / SPLIT / IMPACT_SCAN / PLAN / DOC / REVIEW）：选项 `confirm`（确认进入下一阶段）/ `revise`（提出修改意见）
   - `PLAN` 方案选择：选项 `A` / `B`（/更多方案），不得让用户仅回自由文本
   - `DOC_PENDING_FINAL_CONFIRM` 最终成文门禁：选项 `generate`（确认生成）/ `back`（回到指定阶段修订）
   - 手改检测策略：选项 `adopt`（推荐）/ `merge` / `regenerate`
   - 需求纠偏回退：选项 `rollback-regenerate`（推荐）/ `partial-continue`
   - 阶段跳转入口校验：选项 `补齐后继续` / `回退上一阶段` / `以摘要为临时基线`
   - ❓ 未知项澄清：凡可枚举候选答案的，逐条构造为 question 选项
2. 提问构造要求：
   - 每个决策点一个 question，选项 id/label 与协议语义一一对应；推荐项放第一位并在 label 末尾标注「（推荐）」
   - 同一轮存在多个相互独立的确认点时，合并为一次多 question 调用，减少来回打断
   - 用户经「Other」输入的自由文本，按对应协议处理（如视为修改意见、新路径、新增方案要求等）
3. 模板中的确认块（阶段确认块 / 最终成文门禁块等）仍需输出，作为「待确认内容」的上下文摘要展示；但**决策收集必须走 AskUserQuestion**，二者配合使用而非二选一。
4. 仅当运行环境不存在任何结构化提问工具时，才降级为纯文本确认块交互，此时沿用「Intent normalization」的简短确认语义归一规则。
5. 用户在交互中的选择结果照常写入状态文件（如 `selected_option`、`artifact_dir_confirmed`、手改策略记录），协议其余部分不变。

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

1. `IMPACT_SCAN` 必须评估历史逻辑影响。
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
[START: AUTO|CONTEXT_SCAN|SPEC|SPLIT|IMPACT_SCAN|PLAN|DOC|REVIEW]
-> 版本自检（UPDATE_CHECKING；每会话一次，落后则提示手动更新，失败静默跳过）
-> 入口校验
-> 产物目录确认（仅默认目录：用户确认或改路径）
-> 手改同步（ARTIFACT_SYNCING；每个阶段进入前都重复执行：按 content_sha256 全量比对已落盘产物）
-> CONTEXT_SCAN（无 PRD 依赖，宽而浅）
-> SPEC -> SPLIT -> IMPACT_SCAN（PRD 范围内深扫 + 历史兼容）-> PLAN
-> DOC_FINAL_GATE（必须用户确认）
-> DOC（模板成文）
-> REVIEW
-> (可选) EXPORT（doc/docx）
-> DONE
```

## Non-negotiable rules

1. 默认顺序执行，可显式跳转，但必须通过入口校验。
2. Unknowns 必须显式标记 `❓`，禁止静默假设。
3. 每阶段完成必须落盘并更新 `prd-to-design.state.<lang>.json`；写盘后必须对该产物内容回算 `content_sha256` 写入 `artifacts.<phase>`（手改检测基线），缺失指纹视为违规。
4. `PLAN` 阶段必须给至少两案（A/B）并等待用户选择。
5. 用户未明确选项前，不得进入 `DOC`。
6. 未经最终确认，不得进入 `DOC` 按模板生成正文。
7. 必须先做 `CONTEXT_SCAN`（工程画像，无 PRD 依赖），再读 PRD 形成 `SPEC` / `SPLIT`，再针对切片范围做 `IMPACT_SCAN`（PRD 范围内深扫 + 历史兼容），最后做方案；顺序不可反。
8. 大 PRD 必须先做切片映射并确认，再进入方案与成文。
9. 不得编造接口、表结构、现状能力；必须附证据路径/模块。
10. 图统一使用 `Mermaid`，不得使用 PlantUML；时序/组件图须用 `classDef`/`style` 高亮本期新增/变更并附一行图例。
11. 需求纠偏时必须回退并标记下游产物失效，不得“带病继续”。
12. 涉及字段/类型演进时必须给兼容方案与迁移策略。
13. 命名规范一旦被用户确认，后续阶段必须一致。
14. 历史样本仅用于风格与边界对齐，冲突时以当前 PRD + 现状证据为准。
15. 使用默认（缺省）产物目录时，必须先经用户确认或由用户改路径，确认前不得落盘任何中间产物；用户显式指定 `artifact_dir` / `SDD_ARTIFACT_DIR` 时免确认。
16. 成文必须遵循「成文风格基线（反裹脚布）」：贴代码（业务术语↔代码术语 + 证据）、要点化短句、聚焦变更（图高亮改动点）、敢留白、不注水、同一约束只说一次；详见 [doc.md](doc.md)「成文风格基线」。
17. 所有等待用户确认/选择的门禁点，必须用 `AskUserQuestion`（或环境等价工具）发起结构化交互提问收集决策，不得仅输出文本确认块被动等待；详见「User confirmation interaction」章节。

## State machine

| State | Meaning |
|------|---------|
| `INIT` | 读取输入并检查是否可恢复 |
| `UPDATE_CHECKING` | 启动版本自检（每会话一次）：`fetch` 后比对远程，落后则提示手动更新；非 git/无网络/无 upstream 静默跳过 |
| `ARTIFACT_DIR_CONFIRMING` | 默认产物目录确认（仅 `base_dir_source=default`，确认或改路径） |
| `ENTRY_VALIDATING` | 校验阶段跳转前置条件 |
| `ARTIFACT_SYNCING` | 每个阶段进入前按 `content_sha256` 全量比对已落盘产物，命中手改则接管（adopt/merge/regenerate） |
| `ROLLBACK_SYNCING` | 需求纠偏后的回退与失效标记 |
| `CONTEXT_SCAN_DRAFT` / `CONTEXT_SCAN_CONFIRMING` | 工程画像（无 PRD 依赖，宽而浅）与确认 |
| `SPEC_DRAFT` / `SPEC_CONFIRMING` | 需求分析与确认 |
| `SPLIT_DRAFT` / `SPLIT_CONFIRMING` | 切片映射与确认 |
| `IMPACT_SCAN_DRAFT` / `IMPACT_SCAN_CONFIRMING` | PRD 范围内深扫 + 历史影响评估与确认 |
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
- Artifact Dir Confirmation Block / 默认产物目录确认块
- Phase-jump Entry Validation Block / 阶段跳转入口校验提示块
- Manual Edit Detected Block / 检测到人工修改提示块
- Rollback Prompt Block / 需求纠偏回退提示块

注：以上确认类块仅作「待确认内容」的上下文摘要展示；实际决策收集必须按「User confirmation interaction」章节通过 `AskUserQuestion` 交互完成。

## Phase routing（阶段路由）

### Phase: CONTEXT_SCAN

1. 读取 [context-scan.md](context-scan.md)。
2. 读取 [templates/context-scan.md](templates/context-scan.md) 对应语言骨架。
3. 对 `project_root` / PRD 父目录做轻量盘点（深度 ≤ 2 的目录树、技术栈、入口、关键模块、命名/术语、README/CHANGELOG 摘要），不展开接口签名/表结构。
4. **不**读 PRD（保持无依赖）。
5. 产出并写入 `context.<lang>.md`，进入确认。

### Phase: SPEC

1. 读取 [spec.md](spec.md)。
2. 读取 [templates/spec.md](templates/spec.md) 对应语言骨架。
3. 若存在已确认的 `context.<lang>.md`，先读取并以现状证据校准 SPEC 中的命名、术语、范围可行性与优先级判断。
4. 产出并写入 `spec.<lang>.md`，进入确认。

### Phase: SPLIT

1. 读取 [split.md](split.md)。
2. 读取 [templates/split.md](templates/split.md) 对应语言骨架。
3. 产出并写入 `split.<lang>.md`，进入确认。

### Phase: IMPACT_SCAN

1. 读取 [impact-scan.md](impact-scan.md)。
2. 读取 [templates/impact-scan.md](templates/impact-scan.md) 对应语言骨架。
3. **前置**：以 `context.<lang>.md`（已确认）作为代码现状基线；本阶段在 SPLIT 切片范围内做**深扫**，不重复盘已经在 context 中的浅层信息。
4. 必须覆盖：接口签名、表结构、规则矩阵、历史兼容性。
5. 产出并写入 `impact.<lang>.md`，进入确认。
6. 若存在字段/类型演进，必须增加历史逻辑影响盘点。

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

## Auto-trigger keywords / 自动触发关键词

Cursor 通过 frontmatter 的 `description` 决定是否激活本 skill。下表是触发与排除的真源（source-of-truth），修改 description 时务必同步此处。本 skill 与下游两个 skill（`prd-to-code`、`design-to-code`）共用同一份**互斥分发协议**，三处一致。

### 本 skill 定位

- **输入**：PRD（文本或文件）
- **输出**：系分文档（Mermaid 设计稿、按模板成文），**不写代码**
- **角色场景**：架构师 / 分析师 / 正式流程 / 多系统对齐 / 评审

### Trigger（应触发）

| 类别 | 关键词 |
|------|--------|
| 流程 | `PRD`、`需求文档`、`产品需求`、`系分`、`系分文档`、`系统分析`、`方案设计`、`架构设计`、`概要设计`、`详细设计`、`接口设计`、`走流程`、`正式流程`、`评审`、`研发对齐`、`多系统`、`跨上下游`、`工程画像`、`工程现状`、`仓库摸底`、`两层现状` |
| 阶段 | `CONTEXT_SCAN`、`SPEC`、`SPLIT`、`IMPACT_SCAN`、`PLAN`、`DOC`、`REVIEW`、`A B 方案`、`A/B 方案` |
| 制图 | `Mermaid`、`组件图`、`时序图`、`ER 图`、`类图`、`状态图` |
| 演进 | `字段新增`、`类型新增`、`历史兼容`、`兼容处理`、`迁移策略`、`命名规范` |
| 参数 | `start_phase`、`manual_edit_mode`、`session_id`、`artifact_dir`、`template_path`、`historical_docs_dir`、`export_format` |
| **独占产物**（不与他 skill 撞名） | `split.<lang>.md`、`review.<lang>.md`、`prd-to-design.state.<lang>.json`、`system-analysis.<lang>.md`、`impact.<lang>.md`、`系分模版.md` |
| **共享产物**（与并列 skill 同名，按 skill 子目录区分） | `context.<lang>.md` |
| 操作 | `按模板输出`、`成文`、`导出 doc/docx`、`继续历史` / `resume`、`restart`、`adopt` / `merge` / `regenerate`、`需求纠偏` / `回溯` / `理解错误` |

### Do NOT trigger（应避让）

- 用户只有 PRD 且明确要 **直接写代码 / 个人需求 / 小工具 / 原型 / POC / 跳过系分** → 让 [`prd-to-code`](../prd-to-code/SKILL.md) 接管
- 用户已有系分文档（`prd-to-design.state` / `system-analysis` / `review.md`）且要 **基于系分写代码 / 开发 / 实现** → 让 [`design-to-code`](../design-to-code/SKILL.md) 接管
- 单纯 PRD 解读、撰写、评审，未要求出系分文档 → 不触发

### Disambiguation（互斥分发协议 · 三 skill 一致）

| 用户输入特征 | 路由 |
|--------------|------|
| 仅 PRD + `系分` / `系统分析` / `Mermaid` / `方案设计` / `架构设计` / `概要设计` / `详细设计` / `接口设计` / `A B 方案` / `多系统` / `走流程` / `评审` | **本 skill（#1）** |
| 已有系分文档（`prd-to-design.state` / `system-analysis` / `review.md` / `dev-plan` / `alignment`）+ `开发` / `写代码` / `实现` / `落地` | `design-to-code`（#2） |
| 仅 PRD + `写代码` / `开发` / `实现` + `个人需求` / `小需求` / `小工具` / `内部工具` / `POC` / `原型` / `快速实现` / `不要系分` / `跳过系分` | `prd-to-code`（#3） |
| 仅 PRD + 无场景信号（只说"做一下"/"开始"） | **三方都反问，统一话术见下** |

### Default ask on ambiguity（仅 PRD + 无下游意图时三方一致反问）

> 这个 PRD 你想要：
> - **A. 先生成系分文档**（适合多系统、对齐、正式流程、要架构图）→ `prd-to-design`
> - **B. 直接生成代码**（适合个人需求、小工具、原型、POC、快速实现）→ `prd-to-code`
> - **C. 已有系分文档要写代码** → `design-to-code`

### Trigger examples / 触发示例（正反例）

LLM 路由判断锚点。**新增触发场景前，先在此处加一条正/反例**。

#### ✅ 应触发本 skill（#1）

1. "这是 PRD 文件 `@prd.md`，帮我按系分模板出一份系统分析文档"
2. "PRD 改了，重出方案，给 A/B 两个对比，画一下时序图和组件图"
3. "`@PRD-微粒贷.md` 这是多系统改动，请按 lcs 和 instloancore 分别出系分"
4. "需要给评审会准备方案，PRD 给你了，按 `templates/系分模版.md` 成文"
5. "新增 `product_type` 字段需要兼容历史数据，先出系分文档评估影响"
6. "`@spec.zh-CN.md` 已经确认，进入 SPLIT 阶段"
7. "`@plan.zh-CN.md` 我手改加了方案 B 的成本估算，继续 DOC"
8. "导出 docx 给业务评审"

#### ❌ 不应触发本 skill（应让位）

| Prompt | 该路由到 |
|--------|----------|
| "`@prd.md` 帮我把这个小工具实现一下" | `prd-to-code`（个人小工具，跳过系分） |
| "这是产品需求，帮我直接写代码就行" | `prd-to-code` |
| "PRD 在这里，研发自己用，POC 一下" | `prd-to-code` |
| "`@prd-to-design.state.zh-CN.json` 系分确认了，开始开发" | `design-to-code` |
| "`@review.zh-CN.md` 已经评审完了，按这个开始 coding" | `design-to-code` |
| "`@dev-plan.zh-CN.md` 这版 dev plan 我看一下" | `design-to-code`（涉及它的独占产物） |
| "把这份 PRD 解读一下给我听" | 不触发任何 SDD skill（无下游产出意图） |

## Additional resources

- [context-scan.md](context-scan.md) — CONTEXT_SCAN 工程画像规则（无 PRD 依赖）
- [spec.md](spec.md) — SPEC 阶段规则
- [split.md](split.md) — SPLIT 阶段规则
- [impact-scan.md](impact-scan.md) — IMPACT_SCAN 深扫与历史影响规则
- [plan.md](plan.md) — PLAN 阶段规则
- [doc.md](doc.md) — DOC 阶段规则
- [review.md](review.md) — REVIEW 阶段规则
- [templates/zh-CN.md](templates/zh-CN.md) — 中文模板块（footer/confirm/gate/resume）
- [templates/en.md](templates/en.md) — 英文模板块（footer/confirm/gate/resume）
- [templates/context-scan.md](templates/context-scan.md) — CONTEXT_SCAN 固定骨架
- [templates/spec.md](templates/spec.md) — SPEC 固定骨架
- [templates/split.md](templates/split.md) — SPLIT 固定骨架
- [templates/impact-scan.md](templates/impact-scan.md) — IMPACT_SCAN 固定骨架
- [templates/plan.md](templates/plan.md) — PLAN 固定骨架
- [templates/doc.md](templates/doc.md) — DOC 阶段固定骨架
- [templates/review.md](templates/review.md) — REVIEW 阶段固定骨架
- [templates/系分模版.md](templates/系分模版.md) — 默认系分模板
- [examples.md](examples.md) — 历史样本“输入 PRD -> 输出骨架”示例
