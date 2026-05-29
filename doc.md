# DOC（按模板成文）

本文件仅定义 DOC 阶段规则。固定阶段骨架请使用 `templates/doc.md`，最终正文模板请使用 `templates/系分模版.md`。

## 目标

将已确认的阶段产物（CONTEXT_SCAN/SPEC/SPLIT/IMPACT_SCAN/PLAN）收敛为最终系分文档。

## 进入条件（缺一不可）

1. `CONTEXT_SCAN`、`SPEC`、`SPLIT`、`IMPACT_SCAN`、`PLAN` 已确认并落盘。
2. 用户通过 Final Gate Before DOC（明确回复“确认生成”或同义指令）。
3. 已明确 `template_path`（未指定时默认 `templates/系分模版.md`）。

## 成文规则

- 严格遵循模板章节与顺序，不得改动主结构：
  1. 概述
  2. 方案设计
  3. 数据库设计
  4. 接口设计
  5. 非功能性设计
  6. 其他
- 未知内容必须写 `待确认（原因）`，不得编造。
- 所有图统一使用 `plantuml` 代码块。
- 文档内容必须与已选方案一致。
- 多文档场景下，每份文档都要显式声明系统边界、输入输出、依赖系统。
- 若存在字段/类型演进，正文必须新增“历史逻辑影响与兼容处理”并覆盖：
  - 历史数据处理
  - 旧链路兼容
  - 发布与回滚策略
- 若 PLAN 已确认命名约束，DOC 必须逐项保持一致，不得回退到旧命名。

## 输出与落盘

- 最终文档建议命名：
  - 单文档：`system-analysis.<lang>.md`
  - 多文档：`system-analysis.<system>.<lang>.md`
- 若用户指定 `output_path`，优先按用户路径输出。
- 更新 `artifact_root/prd-to-design.state.<lang>.json`（至少包含 `final_docs` 与 `current_state=DOC_CONFIRMING`）；`final_docs` 每一项需记录 `path`、`content_sha256`、`written_at` 与 `source=skill`，作为手改检测基线。
- 若用户要求导出 `doc/docx`，在 REVIEW 后执行导出并记录导出路径。

## 与下游衔接

DOC 完成后进入 `REVIEW`，执行交付前一致性校验。
