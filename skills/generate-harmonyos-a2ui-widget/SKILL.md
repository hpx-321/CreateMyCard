---
name: generate-harmonyos-a2ui-widget
description: 生成、审查和修复用于 AI 桌面卡片与元服务卡片场景的 HarmonyOS A2UI v0.9 JSONL DSL。用于把任意信息展示、轻操作、列表、仪表、条件状态等需求泛化为 1×2、2×2、2×4、4×4 桌面卡片；同时根据当前 Docs 项目的扩展组件文档和 Schema 处理字段归属、组件名、数据绑定、视觉对比和端侧渲染差异。
---

# 生成 HarmonyOS A2UI 桌面卡片

生成可直接交给 GenUI/A2UIRender 渲染的 A2UI JSONL。本 Skill 已把生成所需的协议、组件字段、布局模板、示例资源和校验脚本收敛到本目录内；日常使用先从 [参考入口](references/README.md) 进入对应模块。

## 核心原则

- **外部开发文档优先。** 当 Skill 内规则与当前 Docs 开发文档、组件 Reference 或 Schema 冲突时，以外部开发文档为准；随后按 [source-decisions.md](references/source-decisions.md) 同步修订本地参考。
- 先抽象需求的信息角色，再选择模板：`primary`（主指标/结论）、`context`（解释/来源/时间）、`details`（2-5 个辅助槽位或列表）、`action`（轻操作）、`state`（加载/空态/异常）。
- **先整体结构排版，后组件内部细节。** 生成 `updateComponents` 前必须先确定卡片尺寸、根容器、主信息区/上下文区/操作区的槽位关系和组件邻接表；只有结构稳定后，才填写每个组件的 `content`、`styles`、事件和数据绑定。
- 示例只提供结构参考，不复用示例业务、文案或固定色值。天气、日程、健康、设备、金融、快递等都应映射到同一套信息角色。
- 不默认生成 2×2。按信息槽位、操作数量、可见文本行、组件估算选择最小可承载尺寸。
- 若端侧对某个组件样式存在版本差异，优先使用更可控的组合结构。例如需要精确 CTA 字色时，可用可点击 `Row` + `Text` 代替原生 `Button`。

## 生成 SOP（强制执行）

1. 从 [references/README.md](references/README.md) 确认阅读路线；若发现与当前外部 Docs 开发文档冲突，按 [source-decisions.md](references/source-decisions.md) 处理。
2. 阅读 [capability.md](references/capability.md) 判断需求是否可满足。不可满足时说明原因和替代方案，停止生成。
3. 阅读 [template.md](references/template.md) 确定尺寸、布局和视觉策略。未指定尺寸时必须按信息量、组件数量、操作数量和排版结构选择最小可承载尺寸。
4. 字段或组件不确定时，按 [extended-component-catalog.md](references/extended-component-catalog.md) → [component-field-reference.md](references/component-field-reference.md) → [common-fields-and-styles.md](references/common-fields-and-styles.md) 查证。
5. 阅读 [guide.md](references/guide.md) 编写 DSL：先画整体布局骨架和扁平组件邻接表，再补组件内部字段与 DataModel，并按 `createSurface` → `updateComponents` → `updateDataModel` 输出。
6. 运行 `python scripts/validate_a2ui_jsonl.py <输出文件>`，修复所有错误；视觉对比失败按高优先级设计错误处理。

## 默认约束

- 协议版本 `v0.9`，Catalog `ohos.a2ui.extended.catalog`。
- 扩展组件优先使用当前聚合 Schema/Reference 中的简名：`Text`、`Button`、`Column`、`Progress`、`Select`、`Tabs`、`TabContent`、`Navigation`/`NavContainer`、`Web` 等；旧式 `Extended.*` 只作为兼容输入处理，不作为新生成默认。
- 输出格式为 JSONL：每行一个完整 JSON 对象，无 Markdown 围栏、注释、尾随逗号。
- 只生成 UI DSL + 代表性初始数据；不虚构数据绑定计划、查询命令、权限或刷新策略。
- 示例数据必须虚构，不暴露真实隐私。
- 字段层级以 [guide.md](references/guide.md) 的兼容矩阵和 [component-field-reference.md](references/component-field-reference.md) 为准；布局字段若存在顶层/`styles` 差异，生成时选择一种并保持一致，不双写。
- 视觉对比问题不是“可选优化”。若 CTA 在局部背景上不清晰，或按钮文字与按钮底色不清晰，视为高优先级错误，生成前必须修正。
- 所有可见层都必须有区分度：文字层清晰可读，区域层与外层背景分离，背景层不能抢占正文与操作区。
- CTA 必须同时通过内对比（按钮文字与按钮底色）和外对比（按钮轮廓与局部背景）。

## 参考文件索引

| 模块 | 文件 | 用途 |
|------|------|------|
| 入口 | [references/README.md](references/README.md) | 阅读路线、模块职责、闭环边界 |
| 能力 | [references/capability.md](references/capability.md) | 能力范围、不支持场景、替代方案 |
| 布局 | [references/template.md](references/template.md) | 尺寸模板、2×2 泛化规律、配色、背景图、反模式 |
| 生成 | [references/guide.md](references/guide.md) | DSL 生成步骤、字段层级、自检清单、修复对照 |
| 协议 | [references/protocol-core.md](references/protocol-core.md) | 消息、邻接表、DataModel、表达式、事件、多设备规则 |
| 通用 | [references/common-fields-and-styles.md](references/common-fields-and-styles.md) | 通用字段、`styles`、背景/图片/渐变/阴影 |
| 选型 | [references/extended-component-catalog.md](references/extended-component-catalog.md) | 常用组件快速索引 |
| 字段 | [references/component-field-reference.md](references/component-field-reference.md) | 组件专有字段定义 |
| 决策 | [references/source-decisions.md](references/source-decisions.md) | 外部开发文档引入策略与冲突优先级 |
| 模板参考 | [references/dsl-templates/](references/dsl-templates/) | 三个 2×2 DSL 模板与资源 |

## 输出约定

- 用户只要 DSL：仅返回 JSONL，不附加解释文字。
- 用户同时要求解释：解释与 JSONL 分开，不混入消息流。
- 有输出路径：写入后执行校验脚本。
