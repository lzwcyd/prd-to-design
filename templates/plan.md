# PLAN 阶段模板

固定文案单一来源：PLAN 阶段输出骨架请仅使用本文件对应语言模板。

## zh-CN

```markdown
# PLAN（方案设计与决策）

## 方案 A
- 业务用例变化：{a_use_case}
- 模块改动：{a_modules}
- 时序变化：{a_sequence}
- 数据影响：{a_data}
- 接口影响：{a_api}
- 非功能影响：{a_nfr}
- 灰度与回滚：{a_gray_rollback}
- 回归范围：{a_regression}

## 方案 B
- 业务用例变化：{b_use_case}
- 模块改动：{b_modules}
- 时序变化：{b_sequence}
- 数据影响：{b_data}
- 接口影响：{b_api}
- 非功能影响：{b_nfr}
- 灰度与回滚：{b_gray_rollback}
- 回归范围：{b_regression}

## 方案对比
| 维度 | 方案 A | 方案 B |
| --- | --- | --- |
| 实现复杂度 | {a_complexity} | {b_complexity} |
| 改造范围 | {a_scope} | {b_scope} |
| 兼容性风险 | {a_risk} | {b_risk} |
| 交付周期 | {a_timeline} | {b_timeline} |
| 可维护性 | {a_maintainability} | {b_maintainability} |

## 推荐结论
{recommendation}

## 命名约束（如用户指定）
- 类名/接口名规范：{naming_convention}
- API 路径规范：{api_path_convention}
- 需替换历史命名：{legacy_name_to_replace}
- 常规流程命名约束：不得使用 `LEGACY`/`历史` 语义命名（如用 `TERM_ASSET_MGMT`，不用 `LEGACY_*`）

## 历史兼容方案（字段/类型演进场景必填）
| 兼容点 | 旧逻辑影响 | 方案A | 方案B | 推荐 |
| --- | --- | --- | --- | --- |
| {compat_item} | {legacy_impact} | {a_compat} | {b_compat} | {compat_recommendation} |

## 需用户决策
- 请选择：方案 A / 方案 B / 变体 C
```

## en

```markdown
# PLAN (Solution Design and Decision)

## Option A
- Use-case changes: {a_use_case}
- Module changes: {a_modules}
- Sequence changes: {a_sequence}
- Data impact: {a_data}
- API impact: {a_api}
- NFR impact: {a_nfr}
- Gray release and rollback: {a_gray_rollback}
- Regression scope: {a_regression}

## Option B
- Use-case changes: {b_use_case}
- Module changes: {b_modules}
- Sequence changes: {b_sequence}
- Data impact: {b_data}
- API impact: {b_api}
- NFR impact: {b_nfr}
- Gray release and rollback: {b_gray_rollback}
- Regression scope: {b_regression}

## Option Comparison
| Dimension | Option A | Option B |
| --- | --- | --- |
| Implementation complexity | {a_complexity} | {b_complexity} |
| Change scope | {a_scope} | {b_scope} |
| Compatibility risk | {a_risk} | {b_risk} |
| Delivery timeline | {a_timeline} | {b_timeline} |
| Maintainability | {a_maintainability} | {b_maintainability} |

## Recommendation
{recommendation}

## Naming Constraints (if user specified)
- Class/API naming convention: {naming_convention}
- API path convention: {api_path_convention}
- Legacy names to be replaced: {legacy_name_to_replace}
- Regular-flow naming rule: do not use `LEGACY`/`history` semantics for active normal flows

## Legacy Compatibility Plan (required for field/type evolution)
| Compatibility Item | Legacy Impact | Option A | Option B | Recommended |
| --- | --- | --- | --- | --- |
| {compat_item} | {legacy_impact} | {a_compat} | {b_compat} | {compat_recommendation} |

## Decision Needed
- Please choose: Option A / Option B / Variant C
```
