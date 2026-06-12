# 协议核心速查

> **用途：** 本文件负责 A2UI v0.9 协议、消息、DataModel、表达式、事件和多设备规则。布局和视觉策略见 [template.md](template.md)，组件字段见 [component-field-reference.md](component-field-reference.md)。

## 1. 消息信封

每条 A2UI 消息是一个完整 JSON 对象，顶层必须包含 `version`，且只能包含一个消息体。

```json
{"version":"v0.9","createSurface":{}}
```

允许的消息体：

| 消息体 | 方向 | 用途 |
|---|---|---|
| `createSurface` | Agent → 端侧 | 创建 UI Surface，并绑定 Catalog |
| `updateComponents` | Agent → 端侧 | 提交或更新扁平组件邻接表 |
| `updateDataModel` | Agent → 端侧 | 写入或更新 Surface 数据模型 |
| `deleteSurface` | Agent → 端侧 | 销毁 Surface |

桌面卡片初始化通常固定输出 3 行：`createSurface` → `updateComponents` → `updateDataModel`。

## 2. createSurface

```json
{"version":"v0.9","createSurface":{"surfaceId":"card-id","catalogId":"ohos.a2ui.extended.catalog","theme":{"primaryColor":"#0A59F7"}}}
```

| 字段 | 必填 | 说明 |
|---|---:|---|
| `surfaceId` | 是 | Surface 唯一标识；同一 JSONL 流中必须一致 |
| `catalogId` | 是 | 桌面卡片默认使用 `ohos.a2ui.extended.catalog` |
| `theme` | 否 | 主题配置，可含 `primaryColor`、`darkPrimaryColor` |
| `sendDataModel` | 否 | 是否在 Action 中附带 DataModel 快照；普通卡片不主动生成 |

## 3. updateComponents

```json
{"version":"v0.9","updateComponents":{"surfaceId":"card-id","components":[{"id":"root","component":"Column","children":["title"]},{"id":"title","component":"Text","content":"Hello"}]}}
```

组件使用扁平邻接表：所有组件都在 `components` 数组中声明，通过 `children`、`child`、`childrenIf`、`childrenElse` 等字段引用组件 id。禁止把组件对象直接嵌套到 `children` 数组里。

邻接表生成规则：

- 必须定义唯一入口组件 `root`。
- 每个组件必须有稳定唯一的 `id` 和合法 `component`。
- 子组件 id 可乱序出现，但最终必须都在同一组件集合或此前更新中定义。
- 同一 Surface 不能混用 Basic Catalog 和鸿蒙扩展 Catalog。
- 扩展 Catalog 中 `Text` 用 `content`，`Image` 用 `src`，`Button` 用 `label`。

## 4. 静态 children 与模板 children

静态子组件：

```json
{"id":"col","component":"Column","children":["title","body","footer"]}
```

模板子组件：

```json
{"id":"list","component":"List","children":{"componentId":"itemTemplate","path":"/items","itemVar":"$item","indexVar":"$index"}}
```

模板规则：

- `componentId` 指向同一组件集合中已定义的模板组件。
- `path` 指向 DataModel 中的数组。
- `itemVar` 默认可用 `$item`，`indexVar` 默认可用 `$index`。
- 卡片中数组内容优先用模板列表，而不是写死多个重复组件。

## 5. updateDataModel

```json
{"version":"v0.9","updateDataModel":{"surfaceId":"card-id","path":"/","value":{"metric":{"valueLabel":"128"}}}}
```

| 字段 | 必填 | 说明 |
|---|---:|---|
| `surfaceId` | 是 | 目标 Surface |
| `path` | 否 | JSON Pointer 路径，默认 `/` |
| `value` | 否 | 新值；省略时表示删除对应 key |

DataModel 规则：

- 组件结构和数据分离：组件定义骨架，DataModel 填内容。
- 初始化时推荐 `path:"/"` 写入完整数据树。
- 所有 `{ "path": "/..." }` 绑定必须在初始 DataModel 中有值。
- 路径使用 JSON Pointer，如 `/user/name`、`/items/0`。
- 展示值和原始值分离，如 `temperatureC:31` 与 `temperatureLabel:"31°"`。
- 示例数据必须虚构，不暴露真实隐私。

## 6. 动态值

组件字段可使用三种来源：

| 类型 | 写法 | 适用 |
|---|---|---|
| 字面量 | `"content":"上海"` | 固定文案 |
| 路径绑定 | `"content":{"path":"/weather/city"}` | 从 DataModel 读取 |
| 表达式 | `"content":"{{ $__DataModel.count + ' 项' }}"` | 简单计算、拼接、条件 |

生成优先级：路径绑定 > 表达式 > 字面量。能直接绑定的不要写复杂表达式。

## 7. 表达式

表达式仅在 `ohos.a2ui.extended.catalog` 下使用，包裹在 `{{ }}` 中。

可用变量：

| 变量 | 说明 |
|---|---|
| `$__DataModel` | 当前 Surface 的完整数据模型 |
| `$__WindowWidthBreakpoint` | 当前窗口断点：`xs`、`sm`、`md`、`lg`、`xl` |
| `$__ColorMode` | 色彩模式：`light`、`dark` |
| `$item` | 模板循环当前项 |
| `$index` | 模板循环索引 |

运算符优先级：

| 优先级 | 运算符 | 说明 |
|---:|---|---|
| 1 | `()` | 分组/函数调用 |
| 2 | `.` `[]` | 成员访问/数组访问 |
| 3 | `!` | 逻辑非 |
| 4 | `*` `/` `%` | 乘、除、取模 |
| 5 | `+` `-` | 加、减；`+` 可做字符串拼接 |
| 6 | `<` `>` `<=` `>=` | 比较 |
| 7 | `==` `!=` | 相等 |
| 8 | `&&` | 逻辑与 |
| 9 | `||` | 逻辑或 |
| 10 | `?:` | 三元条件 |

类型转换：

- `+`：任一操作数为字符串时做字符串拼接，否则数值加法。
- `-`、`*`、`/`：转为数值运算。
- `&&`、`||`、`!`、`?:`：条件转为布尔值。
- 不假设支持任意 JavaScript 方法；内置函数按端侧函数目录和实际 Schema 约束使用。

## 8. EventHandler

事件字段值是 EventHandler 数组，不是单个字符串。

```json
{"onClick":[{"call":"handlePrimaryAction","args":{"targetId":{"path":"/action/targetId"}}}]}
```

| 字段 | 必填 | 说明 |
|---|---:|---|
| `call` | 是 | 函数名；无函数目录时使用语义明确的占位契约名 |
| `args` | 否 | 参数对象，可含路径绑定或表达式 |
| `as` | 否 | 将返回值保存为事件链临时变量 |
| `condition` | 否 | 条件表达式；为 false 时跳过该 handler |

不要虚构系统 API 已存在；需要端侧注册的函数必须保持命名清晰。

## 9. 多设备自适应

断点变量：`$__WindowWidthBreakpoint`。合法值只有 `xs`、`sm`、`md`、`lg`、`xl`。

| 断点 | 宽度范围 |
|---|---|
| `xs` | 0 - 320vp |
| `sm` | 320 - 600vp |
| `md` | 600 - 840vp |
| `lg` | 840 - 1024vp |
| `xl` | 1024vp+ |

单位：

| 单位/值 | 说明 |
|---|---|
| 纯数字 | 默认 vp |
| `"Nvp"` | 虚拟像素 |
| `"Nfp"` | 字体像素；注意本 Skill 生成 `fontSize` 默认用纯数字 |
| `"N%"` | 相对父容器百分比 |
| `"matchParent"` | 填充父容器 |
| `"wrapContent"` | 内容自适应 |

If 组件：

```json
{"id":"adaptive","component":"If","condition":"{{ $__WindowWidthBreakpoint == 'xs' || $__WindowWidthBreakpoint == 'sm' }}","childrenIf":["compact"],"childrenElse":["regular"]}
```

自适应强规则：

1. 断点变量名必须是 `$__WindowWidthBreakpoint`。
2. 不使用 `medium`、`small` 等非标准断点值。
3. `If` 必须同时提供 `childrenIf` 和 `childrenElse`。
4. 两个分支引用的组件 id 必须已定义。
5. `fontSize` 用数字，不写成 `"16fp"` 字符串。
6. `visibility:"none"` 不占位；`"hidden"` 仍占位。
7. `Grid.columnsTemplate` 使用 `"1fr 1fr"`，不使用 `repeat()`。

## 10. deleteSurface 与 Action

普通桌面卡片生成一般不主动输出 `deleteSurface`。如需销毁：

```json
{"version":"v0.9","deleteSurface":{"surfaceId":"card-id"}}
```

用户交互由端侧产生 Action 消息。DSL 生成时只需在组件上声明 `onClick`、`onChange`、`onSelect` 等对应组件支持的事件链，不虚构服务端 Action 回包。
