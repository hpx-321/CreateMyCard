# 开发文档引入决策

> **用途：** 记录本 Skill 曾依赖的 Docs 开发文档，以及为保证目录闭环采用的引入策略。日常生成先看 [README.md](README.md)；后续维护或处理冲突时先更新本文件，再同步对应本地参考文档。

## 决策原则

0. **外部开发文档优先**：若本 Skill 规则与当前 Docs 开发文档、组件 Reference 或 Schema 冲突，以外部开发文档为准，随后同步修订本 Skill。
1. **生成必需且篇幅短的规则**：近全量引入，避免遗漏协议约束。
2. **生成必需但原文很长或包含大量非桌面卡片内容的规则**：精简引入，只保留桌面卡片 DSL 生成会用到的字段、反模式和校验规则。
3. **概念解释、端侧工程代码、示例页面、图片说明**：不引入，仅吸收为本地判断规则。
4. **大 Schema**：不全量复制；提取组件字段、通用样式、事件和模板字段快照，并由校验脚本兜底。

## 外部文档内部冲突优先级

当外部开发文档之间写法不一致时，按下列顺序判断：

1. **组件单页 Reference 优先于聚合 Schema 的字段位置**：例如 Row/Column 的 `justifyContent`、`alignItems`，Grid 的 `columnsTemplate`，List 的 `scrollBar` 在组件单页中是组件顶层字段；生成时按组件单页处理，不双写到 `styles`。
2. **聚合 Schema 优先于旧式 `Extended.*` 页面**：新生成默认使用简名组件；例如简名 `Select` 使用 Schema 中的 `onSelect`，旧 `Extended.Select` 页面中的 `change` 事件仅作为兼容背景。
3. **专项输出规范优先于概念示例**：例如多设备输出规范要求 `fontSize` 使用数字，即使概念示例中出现 `"16fp"`，桌面卡片生成仍使用数字。
4. **运行经验只能补充，不能覆盖文档**：若端侧渲染经验与上述文档冲突，先记录为兼容风险，不把经验写成主规则。

## 引入清单

下表中的“原始开发文档”只作为维护溯源标识，不是生成 DSL 时需要跳转读取的路径。日常生成只读取“本地落点”列中的本目录文件。

| 原始开发文档 | 决策 | 本地落点 | 原因 |
|---|---|---|---|
| `reference/messages.md` | 近全量引入 | [protocol-core.md](protocol-core.md) | 消息信封、四类消息和 `catalogId` 是 DSL 输出硬约束，原文短且高价值。 |
| `concepts/components-and-layout.md` | 精简引入 | [protocol-core.md](protocol-core.md)、[common-fields-and-styles.md](common-fields-and-styles.md) | 只需要邻接表、静态/模板 children、Basic/扩展差异；端侧渲染代码和效果图不需要。 |
| `concepts/data-model-and-binding.md` | 精简引入 | [protocol-core.md](protocol-core.md)、[guide.md](guide.md) | 保留 DataModel、路径绑定、展示值/原始值分离、初始化路径规则；表单示例和端侧代码不引入。 |
| `concepts/expression-language.md` | 精简引入 | [protocol-core.md](protocol-core.md)、[guide.md](guide.md) | 保留 `{{ }}`、变量、运算符、类型转换、可用位置；长示例压缩为规则。 |
| `specification/1_0/harmonyos-extension-protocol.md` | 精简引入 | [protocol-core.md](protocol-core.md)、[common-fields-and-styles.md](common-fields-and-styles.md)、[component-field-reference.md](component-field-reference.md) | 原文是总规范，但本 Skill 已拆成生成规则、字段参考和校验脚本；避免重复全文。 |
| `concepts/multi-device-adaptation.md` | 精简引入 | [protocol-core.md](protocol-core.md)、[template.md](template.md)、[guide.md](guide.md) | 保留单位、断点、If、三种策略；端侧断点获取代码不属于 DSL 生成闭环。 |
| `guides/multi-device-best-practices.md` | 精简引入 | [template.md](template.md)、[guide.md](guide.md) | 保留策略选择、6 种 DSL 模式、Prompt 强化规则；Prompt 构建和端侧错误回调不引入。 |
| `guides/prompts/multi-device/output-spec-rules.md` | 近全量引入 | [guide.md](guide.md)、[protocol-core.md](protocol-core.md) | 8 条规则短且直接防止生成错误。 |
| `reference/extended-components/overview.md` | 精简引入 | [common-fields-and-styles.md](common-fields-and-styles.md) | 原文主要是通用属性和 styles 明细；按桌面卡片需要压缩为速查。 |
| `specification/1_0/schemas/extended_catalog.json` | 精简提取 | [common-fields-and-styles.md](common-fields-and-styles.md)、[component-field-reference.md](component-field-reference.md)、[validate_a2ui_jsonl.py](../scripts/validate_a2ui_jsonl.py) | Schema 约 167KB，不适合全量复制；提取 `CommonStyles`、`EventHandler`、`TemplateChildren` 和 21 个组件字段。 |

## 已处理冲突

| 冲突点 | 外部开发文档依据 | 本 Skill 处理 |
|---|---|---|
| `Select` 事件字段 | 聚合 Schema 中简名 `Select` 使用 `onSelect`；旧 `Extended.Select` 页面描述 `change` 事件 | 新生成使用 `onSelect`；校验脚本将 `Select.onChange` 视为错误；旧事件名只作为兼容背景 |
| Row/Column 对齐字段位置 | 组件单页 Reference 将 `justifyContent`、`alignItems` 定义为组件顶层；聚合 Schema 也出现 `styles` 写法 | 新生成按组件单页写顶层；若修复旧 DSL，允许整体迁移但禁止双写 |
| List 方向/滚动字段位置 | 组件单页 Reference 将 `listDirection`、`scrollBar`、`nestedScroll` 定义为组件顶层；聚合 Schema 也出现 `styles` 写法 | 新生成按组件单页写顶层，并显式设置 `scrollBar` |
| Grid 网格字段位置 | 组件单页 Reference 将 `columnsTemplate`、`rowsTemplate`、`columnsGap`、`rowsGap` 定义为组件顶层；聚合 Schema 也出现 `styles` 写法 | 新生成按组件单页写顶层；禁止 `repeat()` |
| `fontSize` 类型 | 多设备概念示例出现 `"16fp"`，但输出规范和组件 Reference 要求 `fontSize` 为 number | 新生成 `fontSize` 使用数字；校验脚本对非表达式字符串报错 |
| `Extended.*` 与简名组件 | 旧单页可能展示 `Extended.Select` 等；聚合 Schema 使用简名 | 新生成默认简名；旧式只作为兼容输入识别 |
| 主视觉图片资源 | 开发文档支持 `backgroundImage`，新模板使用 `Stack + Image` | 不改写资源路径；复制模板时保留资源相对目录结构。产品/设备主体图优先 `Stack + Image`，装饰底纹才考虑 `backgroundImage` |

## 当前闭环边界

本 Skill 生成桌面卡片 DSL 时只依赖本目录：

- 入口说明：[../SKILL.md](../SKILL.md)
- 能力边界：[capability.md](capability.md)
- 协议核心：[protocol-core.md](protocol-core.md)
- 通用字段/样式：[common-fields-and-styles.md](common-fields-and-styles.md)
- 组件字段：[component-field-reference.md](component-field-reference.md)
- 布局模板：[template.md](template.md)
- 生成指南：[guide.md](guide.md)
- 模板参考：[dsl-templates/](dsl-templates/)
- 校验脚本：[../scripts/validate_a2ui_jsonl.py](../scripts/validate_a2ui_jsonl.py)
