# 鸿蒙扩展组件索引

生成组件前先按本文快速选型，再按实际选用的组件读取 [扩展组件字段参考](component-field-reference.md)。通用字段和样式见 [common-fields-and-styles.md](common-fields-and-styles.md)，协议核心见 [protocol-core.md](protocol-core.md)，完整阅读路线见 [README.md](README.md)。若本地规则与当前 Docs 开发文档冲突，按 [source-decisions.md](source-decisions.md) 的外部文档优先级修订。

本文列出生成桌面卡片时最常用的组件。更新后的聚合 Schema 倾向使用简名组件；新生成 DSL 默认使用简名：

`Button`、`Text`、`TextInput`、`Row`、`Column`、`List`、`Stack`、`Grid`、`Image`、`Divider`、`Toggle`、`Progress`、`Radio`、`Checkbox`、`CheckboxGroup`、`Select`、`Navigation`/`NavContainer`、`Tabs`、`TabContent`、`Web`。

协议还定义了虚拟组件 `If`。旧文档中出现的 `Extended.Select`、`Extended.Tabs`、`Extended.TabContent`、`Extended.Web` 作为兼容输入识别；普通桌面卡片仍应尽量避免导航、标签页和 Web。

## 快速选型

| 组件 | 关键字段 | 说明 |
|---|---|---|
| `Column` | `children`、`itemMargin`、`justifyContent`、`alignItems`、`styles` | 用于根布局和纵向分区；按组件单页 Reference，对齐字段新生成写顶层 |
| `Row` | `children`、`itemMargin`、`justifyContent`、`alignItems`、`wrap`、`styles` | `alignItems` 可用 `top`、`center`、`bottom`；按组件单页 Reference，对齐字段新生成写顶层 |
| `List` | `children`、`space`、`listDirection`、`scrollBar`、`nestedScroll`、`styles` | 明确设置 `scrollBar`；数组使用模板子节点；按组件单页 Reference，方向/滚动字段新生成写顶层 |
| `Grid` | `children`、`columnsTemplate`、`rowsTemplate`、`columnsGap`、`rowsGap`、`styles` | 使用 `"1fr 1fr"`，不要使用 `repeat()`；按组件单页 Reference，网格字段新生成写顶层 |
| `Stack` | `children`、`alignContent`、`styles` | 仅在确实需要层叠时使用 |
| `Text` | `content`、`styles` | `content` 支持字面量、路径绑定和表达式 |
| `Image` | `src`、`styles` | `objectFit`、`aspectRatio` 放在 `styles` 内 |
| `Divider` | `styles.vertical`、`styles.strokeWidth`、`styles.color` | Divider 专属属性全部放在 `styles` 内 |
| `Progress` | `value`、`total`、`styles` | `styles.type` 支持 `linear`、`ring`、`eclipse`、`scaleRing`、`capsule` |
| `Button` | `label`、`enabled`、`action`、`styles` | 原生扩展组件名是 `Button`；若端侧覆盖 `fontColor`，用可点击 `Row` + `Text` 兜底 |
| `Toggle` | `isOn`、`label`、`styles` | 仅在卡片确实需要交互时使用 |
| `If` | `condition`、`childrenIf`、`childrenElse` | 虚拟条件组件 |

## 最小示例

```json
{"id":"taskTitle","component":"Text","content":"{{ $item.title }}","styles":{"fontSize":14,"fontColor":"#E6000000","maxLines":1,"textOverflow":"ellipsis","wordBreak":"breakWord"}}
```

```json
{"id":"weatherIcon","component":"Image","src":{"path":"/weather/icon"},"styles":{"width":40,"height":40,"objectFit":"contain","borderRadius":8,"clip":true}}
```

```json
{"id":"progress","component":"Progress","value":{"path":"/tasks/completed"},"total":{"path":"/tasks/total"},"styles":{"width":"100%","height":6,"color":"#0A59F7","type":"capsule"}}
```

## 常见误写

| 错误写法 | 正确写法 |
|---|---|
| `{"component":"Text","text":"..."}` | `{"component":"Text","content":"..."}` |
| `{"component":"Image","url":"..."}` | `{"component":"Image","src":"..."}` |
| `{"component":"Button","child":"labelText"}` | `{"component":"Button","label":"..."}` |
| `{"component":"Row","justify":"spaceBetween"}` | `{"component":"Row","justifyContent":"spaceBetween"}` |
| `{"component":"Row","align":"center"}` | `{"component":"Row","alignItems":"center"}` |
| `{"component":"Extended.Button"}` | `{"component":"Button"}` |
| `{"component":"Extended.Select"}` | `{"component":"Select"}` |
| `{"styles":{"fontSize":"16fp"}}` | `{"styles":{"fontSize":16}}` |
| `{"columnsTemplate":"repeat(2, 1fr)"}` | `{"columnsTemplate":"1fr 1fr"}` |

部分文档页面仍展示 `Extended.*` 旧式组件名。新生成按当前聚合 Schema 使用简名；旧式写法只作为兼容输入识别，并在 DSL 内保持同一风格。
