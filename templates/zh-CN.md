# zh-CN 输出模板

用于 `lang=zh-CN` 的标准回复结构。主 `SKILL.md` 定义流程，本文件定义可复用提示块。

## 每轮三件套

```markdown
**当前阶段：** ...
**下一步：** ...
**需要用户做什么：** ...
```

## 阶段确认块（CONTEXT_SCAN / SPEC / SPLIT / IMPACT_SCAN / PLAN / REVIEW）

```markdown
-------------------
📌 当前阶段：[CONTEXT_SCAN | SPEC | SPLIT | IMPACT_SCAN | PLAN | REVIEW]

请确认以下内容是否正确：
1. ...
2. ...

回复方式：
- 「确认」/「OK」/「继续」-> 进入下一阶段
- 或给出修改意见 -> 本阶段修订后再确认

注：若仅回复简短确认词，默认按“通过当前阶段确认”处理。
-------------------
```

## DOC 最终成文门禁块

```markdown
-------------------
📌 即将进入 DOC（按模板生成最终系分）

已确认阶段：CONTEXT_SCAN / SPEC / SPLIT / IMPACT_SCAN / PLAN
将使用模板：{template_path}
计划输出：{single_or_multi_docs}

请确认是否开始生成最终系分文档：
- 回复「确认生成」/「开始成文」-> 进入 DOC
- 回复修改意见 -> 回到对应阶段修订
-------------------
```

## 历史恢复提示块

```markdown
检测到历史分析记录（{artifact_root}）：
- 当前状态：{current_state}
- 最近更新时间：{last_updated}

请选择：
1. 继续历史（resume）
2. 从头开始（restart）
```

## 默认产物目录确认块（仅缺省路径）

```markdown
-------------------
📁 产物落盘目录确认（系统推导，非你显式指定）

- PRD/工程上层目录（parent_dir）：{parent_dir}（来源：{parent_dir_source}）
- 默认产物基目录（base_dir）：{base_dir}
- 本次会话产物目录（artifact_root）：{artifact_root}

请确认中间产物是否落盘到以上目录：
- 「确认」/「OK」/「继续」-> 沿用该默认目录
- 直接给出新路径 -> 改用你指定的目录（整体替换，不再追加 skill 名）

注：确认前不会创建目录或写入任何中间产物。
-------------------
```

## 阶段跳转入口校验提示块

```markdown
你指定从 `{start_phase}` 开始。

入口校验结果：
- 缺失上游产物：{missing_artifacts}
- 可直接继续：{can_continue}

请选择下一步：
1. 补齐缺失产物后继续
2. 回退到 `{fallback_phase}`
3. 以你提供的摘要作为临时基线继续
```

## 检测到人工修改提示块

```markdown
检测到你手工修改了 `{artifact_path}`（较上次记录已变化）。

请选择处理策略：
1. adopt：采用手改版本继续
2. merge：与当前草稿合并
3. regenerate：基于最新输入重生成该阶段

注：若仅回复「OK/确认/继续」但未指明策略，默认按 adopt 处理。
```

## 需求纠偏回退提示块

```markdown
检测到需求纠偏信号：{rollback_trigger}
建议回退阶段：{rollback_to_phase}
受影响产物：{stale_artifacts}

请确认：
1. 回退并重生成（推荐）
2. 仅局部修订并继续（需说明风险）
```

## 产物文件命名

- `spec.zh-CN.md`
- `split.zh-CN.md`
- `context.zh-CN.md`
- `impact.zh-CN.md`
- `plan.zh-CN.md`
- `review.zh-CN.md`
- `analysis.state.zh-CN.json`

## 阶段骨架模板

- SPEC：`templates/spec.md`（`zh-CN` 区块）
- SPLIT：`templates/split.md`（`zh-CN` 区块）
- CONTEXT_SCAN：`templates/context-scan.md`（`zh-CN` 区块）
- IMPACT_SCAN：`templates/impact-scan.md`（`zh-CN` 区块）
- PLAN：`templates/plan.md`（`zh-CN` 区块）
- DOC：`templates/doc.md`（`zh-CN` 区块）
- REVIEW：`templates/review.md`（`zh-CN` 区块）
