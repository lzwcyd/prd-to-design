# SPEC（意图定义）

本文件仅定义 SPEC 阶段规则。固定输出骨架请使用 `templates/spec.md`，通用提示块请使用 `templates/zh-CN.md` 或 `templates/en.md`。

## 目标

明确“做什么 + 为什么做”，形成后续拆分与方案设计的统一需求基线。

## 阶段规则

- 必须覆盖：业务背景、用户问题、核心目标、范围（In/Out of Scope）、输入输出、验收标准。
- 必须包含：用户价值与优先级判断。
- 所有未知项必须标记为 `❓`，并向用户澄清。
- 阻塞性问题未澄清前，不得推进到 `SPLIT`。
- 若从后续阶段回退到 `SPEC`，需要重建完整基线并重新确认。
- 若用户明确指出“理解错误/范围修正”，必须重写 In/Out of Scope 并标记下游产物待重生成。

## 输出与落盘

1. 按当前 `lang` 使用 `templates/spec.md` 对应语言骨架组织输出。
2. 写入 `artifact_root/spec.<lang>.md`。
3. 更新 `artifact_root/prd-to-design.state.<lang>.json`（至少包含 `current_state`、`last_updated`、`artifacts.spec`）。

## 最小校验清单

- 是否有明确业务目标与成功标准
- 是否列明范围边界（In/Out）
- 是否存在未澄清的阻塞项
- 是否完成阶段确认块并得到用户确认

## 与下游衔接

SPEC 确认后进入 `SPLIT`，由 `split.md` 接管切片和输出规划。
