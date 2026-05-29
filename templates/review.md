# REVIEW 阶段模板

固定文案单一来源：REVIEW 阶段输出骨架请仅使用本文件对应语言模板。

## zh-CN

```markdown
# REVIEW（交付前校验）

## 校验清单
- [ ] 模板章节与顺序完整
- [ ] PRD 核心需求与验收点覆盖
- [ ] 现状证据与改造结论一致
- [ ] 方案选择与正文一致
- [ ] 命名约束（类名/方法名/API）全链路一致
- [ ] 全部图为 Mermaid 且语义一致
- [ ] 接口/数据库/非功能形成闭环
- [ ] 字段/类型演进具备“兼容策略 + 回归验证”闭环
- [ ] 大 PRD 追踪映射完整
- [ ] 多文档联调边界完整

## 缺口与建议
| 编号 | 缺口描述 | 影响范围 | 建议修订 |
| --- | --- | --- | --- |
| G1 | {gap_desc} | {impact_scope} | {suggestion} |

## 交付结论
- 结论：可交付 / 需修订
- 若需修订，回退阶段：{fallback_phase}
- 备注：{notes}
- 导出状态（可选）：{export_status}
```

## en

```markdown
# REVIEW (Pre-delivery Validation)

## Validation Checklist
- [ ] Template chapters and order are complete
- [ ] Core PRD requirements and acceptance points are covered
- [ ] Baseline evidence matches change conclusions
- [ ] Selected option matches final documents
- [ ] Naming constraints (class/method/API) are consistent end-to-end
- [ ] All diagrams are Mermaid and semantically correct
- [ ] API/DB/NFR design forms a closed loop
- [ ] Field/type evolution has a closed loop: compatibility + regression validation
- [ ] PRD-to-section traceability is complete (for large PRD)
- [ ] Cross-doc integration boundaries are complete (for multi-doc output)

## Gaps and Suggestions
| ID | Gap Description | Impact Scope | Suggested Fix |
| --- | --- | --- | --- |
| G1 | {gap_desc} | {impact_scope} | {suggestion} |

## Delivery Decision
- Decision: deliverable / needs revision
- If revision is needed, fallback phase: {fallback_phase}
- Notes: {notes}
- Export status (optional): {export_status}
```
