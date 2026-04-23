# prd-to-design

基于 PRD + 工程现状，按 SDD 流程产出系分文档（设计文档）的 Skill。

> 历史名：`prd-to-system-analysis` / 仓库 `prd-to-system-analysis-skill`。新名 `prd-to-design` 与并列两个 skill `design-to-code`、`prd-to-code` 形成"输入→输出"对称命名。

## 文件结构

- `SKILL.md`：主流程（状态机总控 + 阶段路由）
- 阶段规则：
  - `spec.md`
  - `split.md`
  - `current-state.md`
  - `plan.md`
  - `doc.md`
  - `review.md`
- 模板目录 `templates/`：
  - 通用提示块：`zh-CN.md`、`en.md`
  - 阶段骨架：`spec.md`、`split.md`、`current-state.md`、`plan.md`、`doc.md`、`review.md`
  - 最终系分模板：`系分模版.md`
- `examples.md`：历史样本“输入 PRD -> 输出骨架”示例

## 默认行为（SDD 状态机）

- 默认模板：`templates/系分模版.md`
- 分阶段：`SPEC -> SPLIT -> CURRENT_STATE -> PLAN -> DOC -> REVIEW`
- 中间产物必须落盘：`spec/split/current-state/plan/review` + `analysis.state.<lang>.json`
- 默认产物目录：`<base_dir>/<session_id>/`，其中 `base_dir` 解析优先级：
  1. 显式指定 `artifact_dir`（或环境变量 `SDD_ARTIFACT_DIR`）— 最高
  2. PRD 文件所在目录（PRD 为文件路径时）
  3. 工作目录 / `project_root`（PRD 为内联文本时）
- 大 PRD：先 `SPLIT`（切片映射）并确认，再进入后续阶段
- 方案阶段：必须给至少 A/B 两案并等待用户选择
- `DOC` 之前有最终门禁：用户明确确认后才能按模板生成系分正文
- 图形语法：统一 `PlantUML`
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

## 备注

- 若信息不足，文档中应标注 `待确认（原因）`，不得编造。
- 更多输出骨架请看 `examples.md`。
