# DOC 阶段模板

固定文案单一来源：DOC 阶段输出骨架请仅使用本文件对应语言模板。最终正文必须遵循 `templates/系分模版.md`。

## zh-CN

```markdown
# DOC（按模板成文）

## 成文输入
- 选定方案：{selected_option}
- 模板路径：{template_path}
- 输出模式：{single_or_multi_docs}

## 输出文档计划
| 文档名 | 系统边界 | 输出路径 | 状态 |
| --- | --- | --- | --- |
| {doc_name} | {system_boundary} | {output_path} | 待生成/已生成 |

## 成文约束确认
- 模板章节顺序固定：概述 -> 方案设计 -> 数据库设计 -> 接口设计 -> 非功能性设计 -> 其他
- 图语法：Mermaid
- 未知项：待确认（原因）
- 命名一致性：与 PLAN 选定命名保持一致

## 历史逻辑影响与兼容处理（字段/类型演进场景必填）
- 历史数据处理：{historical_data_strategy}
- 旧链路兼容：{legacy_flow_compatibility}
- 发布/回滚策略：{release_rollback_strategy}

## 生成摘要
{generation_summary}
```

## en

```markdown
# DOC (Template-based Drafting)

## Drafting Inputs
- Selected option: {selected_option}
- Template path: {template_path}
- Output mode: {single_or_multi_docs}

## Output Document Plan
| Document Name | System Boundary | Output Path | Status |
| --- | --- | --- | --- |
| {doc_name} | {system_boundary} | {output_path} | pending/generated |

## Drafting Constraints
- Fixed chapter order: Overview -> Solution Design -> Database Design -> API Design -> Non-functional Design -> Others
- Diagram syntax: Mermaid
- Unknowns: TBD (with reason)
- Naming consistency: must match selected PLAN naming

## Legacy Logic Impact and Compatibility (required for field/type evolution)
- Historical data strategy: {historical_data_strategy}
- Legacy flow compatibility: {legacy_flow_compatibility}
- Release/rollback strategy: {release_rollback_strategy}

## Generation Summary
{generation_summary}
```
