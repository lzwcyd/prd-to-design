# SPLIT（切片与输出规划）

本文件仅定义 SPLIT 阶段规则。固定输出骨架请使用 `templates/split.md`，通用提示块请使用 `templates/zh-CN.md` 或 `templates/en.md`。

## 目标

将 PRD 拆成可独立设计与交付的需求切片，明确单文档/多文档产出策略。

## 阶段规则

- 大 PRD 必须先分段读取并建立目录索引，避免漏读附录和约束。
- 按“业务场景/期数/系统边界”切片。
- 输出 `需求切片映射表`，至少包含：
  - PRD 章节
  - 场景/能力点
  - 目标系统
  - 变更类型（复用/改造/新增）
  - 预计输出文档
- 对每个切片补充“外部触发方/调用边界”与“模板或配置依赖”确认位，避免在 PLAN/DOC 阶段才暴露依赖缺口。
- 若涉及多系统，默认多文档输出（每份文档仅覆盖单系统边界）。
- 用户确认切片映射前，不得推进到 `IMPACT_SCAN`。

## 历史样本对齐

- 若存在历史 PRD/系分样本，先提取系统切分方式和文档颗粒度。
- 历史样本仅用于风格对齐，和当前 PRD 冲突时以当前 PRD 为准。

## 输出与落盘

1. 按当前 `lang` 使用 `templates/split.md` 对应语言骨架组织输出。
2. 写入 `artifact_root/split.<lang>.md`。
3. 更新 `artifact_root/prd-to-design.state.<lang>.json`（至少包含 `artifacts.split`）。
4. 写盘后对 `split.<lang>.md` 的最终内容回算 `SHA-256`，写入 `artifacts.split.content_sha256`、`written_at` 与 `source=skill`，作为后续手改检测基线。

## 最小校验清单

- 切片是否覆盖 PRD 主体需求
- 多系统边界是否清晰、无重叠
- 每个切片是否有明确输出文档归属
- 是否完成阶段确认并得到用户同意

## 与下游衔接

SPLIT 确认后进入 `IMPACT_SCAN`，在切片范围内深扫现状并映射“复用/改造/新增/缺失”，同时评估历史影响。
