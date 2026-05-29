# REVIEW（交付前校验）

本文件仅定义 REVIEW 阶段规则。固定输出骨架请使用 `templates/review.md`，通用提示块请使用 `templates/zh-CN.md` 或 `templates/en.md`。

## 目标

在交付前完成结构、内容、证据、风险的一致性检查，确保文档可评审、可落地、可追溯。

## 阶段规则

- 必须检查：
  - 模板章节与顺序是否完整
  - PRD 核心需求与验收点是否覆盖
  - 工程现状证据与“复用/改造/新增/缺失”是否一致
  - 方案选择与最终正文是否一致
  - 命名约束（如类名/方法名/API 路径）是否全链路一致
  - 图是否全部为 Mermaid 且语义与正文一致
  - 接口、数据库、非功能设计是否形成闭环
  - 字段/类型演进是否给出“兼容策略 + 回归验证”闭环
  - 大 PRD 是否保留“PRD 段落 -> 系分章节”追踪映射
  - 多文档是否补齐跨文档依赖与联调边界
- 若存在缺口，必须先列出缺口和建议，再请求用户确认是否交付。

## 输出与落盘

1. 按当前 `lang` 使用 `templates/review.md` 对应语言骨架组织输出。
2. 写入 `artifact_root/review.<lang>.md`。
3. 更新 `artifact_root/prd-to-design.state.<lang>.json`（至少包含 `artifacts.review` 与 `delivery_status`）。
4. 写盘后对 `review.<lang>.md` 的最终内容回算 `SHA-256`，写入 `artifacts.review.content_sha256`、`written_at` 与 `source=skill`，作为后续手改检测基线。

## 交付结论

- 若用户确认交付：进入 `DONE`。
- 若用户要求修订：回退到对应阶段（通常是 `PLAN` 或 `DOC`）并重新确认。
- 若用户要求导出 `doc/docx`：在 `DONE` 前完成导出并回填输出路径。
