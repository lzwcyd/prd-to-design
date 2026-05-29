# prd-to-design

基于 PRD + 工程现状，按 SDD 流程产出系分文档（设计文档）的 Skill。

> `prd-to-design` 与并列的 `design-to-code`、`prd-to-code` 两个 skill 形成"输入→输出"对称命名。

## 文件结构

- `SKILL.md`：主流程（状态机总控 + 阶段路由）
- 阶段规则：
  - `context-scan.md`（**新增**：CONTEXT_SCAN 工程画像，无 PRD 依赖）
  - `spec.md`
  - `split.md`
  - `impact-scan.md`（PRD 范围内深扫 + 历史影响）
  - `plan.md`
  - `doc.md`
  - `review.md`
- 模板目录 `templates/`：
  - 通用提示块：`zh-CN.md`、`en.md`
  - 阶段骨架：`context-scan.md`、`spec.md`、`split.md`、`impact-scan.md`、`plan.md`、`doc.md`、`review.md`
  - 最终系分模板：`系分模版.md`
- `examples.md`：历史样本“输入 PRD -> 输出骨架”示例

## 默认行为（SDD 状态机）

- 默认模板：`templates/系分模版.md`
- 分阶段：`CONTEXT_SCAN -> SPEC -> SPLIT -> IMPACT_SCAN -> PLAN -> DOC -> REVIEW`
- 中间产物必须落盘：`context/spec/split/impact/plan/review` + `prd-to-design.state.<lang>.json`
- 默认产物目录：`<base_dir>/<session_id>/`，其中 `base_dir` 解析规则（避免与并列 skill `prd-to-code`、`design-to-code` 冲突）：
  1. 显式指定 `artifact_dir`（或环境变量 `SDD_ARTIFACT_DIR`）— 最高，按用户指定路径整体替换
  2. 缺省默认：`<parent_dir>/prd-to-design/`
     - `parent_dir` 优先取 PRD 文件所在目录；PRD 为内联文本时取 `project_root` / 工作目录
     - 强制追加 `prd-to-design/` skill 子目录，避免覆盖其它 skill 同名中间文件
     - ⚠️ 缺省目录为系统推导，落盘前需经用户确认（确认沿用 or 提供新路径）；显式指定 `artifact_dir` / `SDD_ARTIFACT_DIR` 时免确认
- 大 PRD：先 `SPLIT`（切片映射）并确认，再进入后续阶段
- 方案阶段：必须给至少 A/B 两案并等待用户选择
- `DOC` 之前有最终门禁：用户明确确认后才能按模板生成系分正文
- 图形语法：统一 `Mermaid`
- 支持恢复历史（resume/restart）、阶段跳转校验、用户手改中间文档同步（adopt/merge/regenerate）
- 支持需求纠偏回退（范围修正后自动标记下游产物 stale 并重生成）
- 支持字段/类型演进闭环（现状识别 -> 方案兼容 -> 正文落盘 -> REVIEW 校验）
- 支持命名约束贯穿（PLAN 命名清单自动要求同步到 DOC/REVIEW）
- 支持交付导出（用户明确要求时导出 `doc/docx`）

## 最小使用示例

```text
请使用 prd-to-design skill 生成系分：
- prd: /path/to/【PRD】xxx.md
- project_root: /path/to/project
- historical_docs_dir: /path/to/history_docs
- artifact_dir: /path/to/artifacts (可选，作为中间产物基础目录)
- session_id: <当前会话唯一ID> (可选，未指定时使用运行时会话 ID)
```

## 常见场景

### 1) 微粒贷（大 PRD，多系统）

```text
请用该 skill 处理【PRD】微粒贷.md：
先按系统边界拆分（如 lcs / instloancore），
给出 SPLIT 映射表后再继续方案对比和文档产出。
```

### 2) 费率计算器升级（规则密集）

```text
请用该 skill 处理【PRD】费率计算器升级.md：
重点输出规则矩阵（Xcap、未来预估应收、预估减免等）、
灰度开关和兼容策略，再生成系分正文。
```

## 状态机（State Machine）

### 阶段流程图

```mermaid
flowchart LR
    init[INIT] --> sync[ARTIFACT_SYNCING]
    sync -. 每阶段前重复 .-> sync
    sync --> contextScan[CONTEXT_SCAN]
    contextScan --> spec[SPEC]
    spec --> split[SPLIT]
    split --> impactScan[IMPACT_SCAN]
    impactScan --> plan[PLAN]
    plan --> docGate{{DOC_FINAL_GATE}}
    docGate --> doc[DOC]
    doc --> review[REVIEW]
    review --> exportNode[EXPORT 可选]
    exportNode --> doneNode[DONE]
```

### 阶段说明

| 阶段 | 输入 | 产物 | 是否需用户确认 | 入口前置 |
|------|------|------|----------------|----------|
| `CONTEXT_SCAN` | 工程根目录（**不**读 PRD） | `context.<lang>.md` | 是 | 无 |
| `SPEC` | PRD + `context.<lang>.md` | `spec.<lang>.md` | 是 | 有可解析 PRD |
| `SPLIT` | `spec.<lang>.md` | `split.<lang>.md`（切片映射） | 是 | SPEC 已确认 |
| `IMPACT_SCAN` | `split.<lang>.md` + `context.<lang>.md` | `impact.<lang>.md`（PRD 范围内深扫 + 历史影响） | 是 | SPLIT 已确认 |
| `PLAN` | `impact.<lang>.md` | `plan.<lang>.md`（A/B 方案） | 是 + 至少一个用户选定方案 | IMPACT_SCAN 已确认 |
| `DOC` | 全部上游 + `templates/系分模版.md` | `system-analysis.<lang>.md`（按模板成文） | 强门禁：必须通过 `DOC_FINAL_GATE` | PLAN 已确认 |
| `REVIEW` | 最终系分文档 | `review.<lang>.md`（一致性 + 命名 + 兼容性闭环校验） | 是 | DOC 已落盘 |
| `EXPORT`（可选） | `system-analysis.<lang>.md` | `system-analysis.<lang>.docx/.doc` | 仅用户明确要求时执行 | REVIEW 已确认 |

### 状态枚举

| State | 含义 |
|-------|------|
| `INIT` | 读取输入并检查可恢复 |
| `ARTIFACT_DIR_CONFIRMING` | 默认产物目录确认（仅缺省路径，确认或改路径） |
| `ENTRY_VALIDATING` | 校验阶段跳转前置 |
| `ARTIFACT_SYNCING` | 每阶段进入前按 `content_sha256` 全量比对已落盘产物，命中手改则接管（adopt/merge/regenerate） |
| `ROLLBACK_SYNCING` | 需求纠偏后的回退与失效标记 |
| `CONTEXT_SCAN_DRAFT` / `CONTEXT_SCAN_CONFIRMING` | 工程画像与确认 |
| `SPEC_DRAFT` / `SPEC_CONFIRMING` | 需求分析与确认 |
| `SPLIT_DRAFT` / `SPLIT_CONFIRMING` | 切片映射与确认 |
| `IMPACT_SCAN_DRAFT` / `IMPACT_SCAN_CONFIRMING` | PRD 范围内深扫与历史影响 |
| `PLAN_DRAFT` / `PLAN_CONFIRMING` | A/B 方案与用户选择 |
| `DOC_PENDING_FINAL_CONFIRM` | 等待最终成文强门禁 |
| `DOC_DRAFT` / `DOC_CONFIRMING` | 按模板成文与确认 |
| `REVIEW_DRAFT` / `REVIEW_CONFIRMING` | 一致性校验与交付确认 |
| `EXPORT_DRAFT` / `EXPORT_CONFIRMING` | 可选 doc/docx 导出 |
| `DONE` | 终态 |

### 特殊行为

- **默认产物目录确认**：当 `base_dir` 走缺省默认（未显式指定 `artifact_dir` / `SDD_ARTIFACT_DIR`）时，落盘前先经用户确认或改路径；确认前不创建目录、不写入任何中间产物。
- **手改同步**：`manual_edit_mode=on` 时，每阶段前进入 `ARTIFACT_SYNCING`，按 `content_sha256` 对所有已落盘产物做手改检测；命中变化提示 `adopt` / `merge` / `regenerate`。
- **需求纠偏回退**：当用户出现"理解错误 / 回溯 / 改需求 / 范围变更"等信号，识别受影响最早阶段（通常 `SPEC` 或 `SPLIT`），下游产物标记 `stale_due_to_spec_change` 并按新基线重生。
- **DOC 强门禁**：在生成模板正文前必须用户确认通过 `DOC_FINAL_GATE`，否则禁止进入 `DOC`。
- **首次会话默认**：`start_phase=AUTO` 解析为 `CONTEXT_SCAN`；存在已确认 `context.<lang>.md` 时回落到 `SPEC`。

## 备注

- 若信息不足，文档中应标注 `待确认（原因）`，不得编造。
- 更多输出骨架请看 `examples.md`。
