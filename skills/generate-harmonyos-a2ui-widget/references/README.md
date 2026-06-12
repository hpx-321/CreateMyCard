# Skill 参考入口

> **用途：** 本文件是 `generate-harmonyos-a2ui-widget` 的本地参考地图。生成 DSL 时先按阅读路线进入对应模块，避免在多个文档之间重复查找。

## 阅读路线

### 生成新 DSL

1. [capability.md](capability.md)：判断需求是否在能力范围内。
2. [template.md](template.md)：选择尺寸、布局骨架、配色和背景策略。
3. [guide.md](guide.md)：先确定整体结构排版和组件邻接表，再补组件内部细节，最后按协议顺序生成 JSONL 并完成自检。
4. [../scripts/validate_a2ui_jsonl.py](../scripts/validate_a2ui_jsonl.py)：校验输出文件。

### 字段或组件不确定

1. [extended-component-catalog.md](extended-component-catalog.md)：先做组件选型。
2. [component-field-reference.md](component-field-reference.md)：再查组件专有字段。
3. [common-fields-and-styles.md](common-fields-and-styles.md)：补充通用字段、样式、渐变、阴影和资源路径规则。

### 协议、数据和事件不确定

阅读 [protocol-core.md](protocol-core.md)。它负责消息信封、`createSurface`、`updateComponents`、`updateDataModel`、DataModel、表达式、事件和多设备规则。

### 维护或处理冲突

阅读 [source-decisions.md](source-decisions.md)。若本 Skill 与当前外部 Docs 开发文档冲突，以外部开发文档为准，然后同步更新本地参考文件。

## 模块职责

| 文件 | 负责 | 不负责 |
|---|---|---|
| [../SKILL.md](../SKILL.md) | 入口、执行流程、默认约束、输出约定 | 组件字段明细和长篇模板细节 |
| [capability.md](capability.md) | 能力边界、支持/不支持场景、拒绝与替代方案 | 具体布局、字段层级 |
| [template.md](template.md) | 尺寸推断、布局骨架、2×2 泛化规律、配色和背景图策略 | 协议信封、组件字段全集 |
| [guide.md](guide.md) | 生成步骤、自检清单、常见误写修复、最小示例 | 维护来源决策、完整组件目录 |
| [protocol-core.md](protocol-core.md) | A2UI v0.9 消息、邻接表、DataModel、表达式、事件、多设备规则 | 视觉布局策略 |
| [common-fields-and-styles.md](common-fields-and-styles.md) | 通用字段、通用 `styles`、背景/图片/渐变/阴影速查 | 组件专有字段 |
| [extended-component-catalog.md](extended-component-catalog.md) | 常用组件快速选型、最小片段、常见误写 | 完整字段说明 |
| [component-field-reference.md](component-field-reference.md) | 组件专有字段、枚举、兼容说明 | 通用样式全集 |
| [source-decisions.md](source-decisions.md) | 外部开发文档引入策略、冲突优先级、闭环边界 | 日常生成流程 |
| [dsl-templates/](dsl-templates/) | 三个 2×2 参考 DSL 和相关资源 | 可直接复制的业务文案或固定色板 |

## 闭环边界

- 日常生成、审查和修复只依赖本 Skill 目录内文件。
- `dsl-templates/` 中的资源路径按模板相对目录保持一致，不为了闭环改写 DSL 内资源路径。
- 文档内链接应保持相对路径，并指向本目录内文件或子目录。
- 外部 Docs 开发文档是冲突裁决来源；发生冲突时，先按外部文档修订本地参考，再继续生成。

## 校验入口

在技能根目录 `generate-harmonyos-a2ui-widget` 下运行：

```powershell
python scripts/validate_a2ui_jsonl.py <JSONL文件路径>
```

模板 DSL 也应通过同一脚本校验，保证示例与规则一致。
