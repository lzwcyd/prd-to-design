# CONTEXT_SCAN（工程画像 · 无 PRD 依赖 · 宽而浅）

本文件只定义 CONTEXT_SCAN 阶段规则；固定输出骨架请使用 `templates/context-scan.md`，通用提示块请使用 `templates/zh-CN.md` 或 `templates/en.md`。

## 目标

在尚未读取 PRD 的前提下，对当前工程做一次"宽而浅"的画像，给后续 SPEC / SPLIT 与下游 `IMPACT_SCAN` 提供：

- 现有命名 / 术语 / 业务 prefix
- 现有技术栈 / 构建启动方式
- 现有模块清单与入口位置
- 接口/数据层的"线索"（不展开签名/结构 → 留给 `IMPACT_SCAN`）
- 文档证据（README / CHANGELOG / 接口文档目录）

> 配合 `IMPACT_SCAN`（PRD 范围内深扫）形成"两层现状"：CONTEXT_SCAN 摸全局家底；IMPACT_SCAN 在切片范围内做深扫与历史影响评估。

## 阶段规则

- 必须基于工程证据，不得凭空假设。
- 必须"宽而浅"：覆盖工程整体形状，但**不**展开实现细节。
- **不**读 PRD（保持无依赖）；**不**做需求-模块映射；**不**做历史影响评估（留给 `IMPACT_SCAN`）。
- 必须读取的对象（按存在与否取舍）：
  - `README` / `README.md` / `README.<lang>.md`
  - `CHANGELOG.md`
  - 包/依赖清单：`package.json` / `pyproject.toml` / `requirements*.txt` / `Cargo.toml` / `go.mod` / `pom.xml` / `build.gradle*`
  - 顶层目录与一级子目录（深度 ≤ 2）
  - 主要入口文件（`main.*` / `cli.*` / `index.*` / `app.*` / `server.*` 等）
  - 接口/服务目录线索（`controller/` / `api/` / `routes/` / `handler/` / `rpc/` 等仅列路径）
  - 数据层线索（`migrations/` / `entity/` / `model/` / `schema/` / `dao/` 等仅列路径）
- 证据必须可追溯到具体路径。

## 输出与落盘

1. 按当前 `lang` 使用 `templates/context-scan.md` 对应语言骨架组织输出。
2. 写入 `artifact_root/context.<lang>.md`。
3. 更新 `artifact_root/prd-to-design.state.<lang>.json`（至少包含 `artifacts.context`、`current_state`、`last_updated`）。

## 最小校验清单

- 是否给出技术栈与构建启动方式
- 是否给出顶层目录树（深度 ≤ 2）
- 是否列出关键模块 / 入口
- 是否列出接口/数据层的目录线索（仅路径）
- 是否摘录可观察的命名 / 术语
- 证据是否可追溯到路径
- 是否完成阶段确认块并得到用户确认

## 与下游衔接

CONTEXT_SCAN 确认后进入 `SPEC`：

- `SPEC` 必须引用 `context.<lang>.md` 中的命名/术语，发现 PRD 命名冲突时显式提示。
- `SPLIT` 切片可参考 context 中的模块边界初步对齐。
- `IMPACT_SCAN` 阶段在切片范围内做深扫，**不**重复 context 已盘的浅层信息。
