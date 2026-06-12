# 组件通用字段与样式

> **用途：** 本文件只负责所有组件共享的字段、通用 `styles`、背景/图片/渐变/阴影速查和 Schema 快照。组件专有字段见 [component-field-reference.md](component-field-reference.md)，协议消息见 [protocol-core.md](protocol-core.md)。

## 通用字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---:|---|
| `id` | string | 是 | 组件唯一标识，同一 Surface 内不可重复 |
| `component` | string | 是 | 组件类型，如 `Text`、`Column`、`Stack` |
| `styles` | object | 否 | 通用样式对象 |
| `accessibility` | object | 否 | 无障碍信息 |
| `onClick` | EventHandler[] | 否 | 点击事件链 |
| `onAppear` | EventHandler[] | 否 | 出现事件链 |

`accessibility` 可含：

| 字段 | 类型 | 说明 |
|---|---|---|
| `label` | string | 屏幕阅读器朗读的名称或用途 |
| `description` | string | 进一步描述内容或操作结果 |

## 通用 styles 清单

| 样式 | 类型 | 桌面卡片生成要点 |
|---|---|---|
| `width` / `height` | number/string | 数字默认 vp；也可用 `"100%"`、`"matchParent"`、`"wrapContent"` |
| `constraintSize` | object | 限制 `minWidth`、`maxWidth`、`minHeight`、`maxHeight` |
| `margin` / `padding` | number/string/object | 数字或单位字符串；对象键为 `top`、`right`、`bottom`、`left` |
| `borderRadius` | number/string/object | 圆角；图片/渐变根容器配合 `clip:true` |
| `borderWidth` / `borderColor` | number/string | `borderColor` 支持 `#RRGGBB` / `#AARRGGBB` |
| `backgroundColor` | string | 背景色；半透明推荐 `#AARRGGBB` |
| `backgroundImage` | string | 背景图片路径；产品/设备主视觉更推荐 `Stack + Image` |
| `backgroundImageSizeWithStyle` | string/object | `cover`、`contain`、`auto`、`fill` 或 `{width,height}` |
| `linearGradient` | object | 渐变配置；卡片优先使用 `direction + colors` |
| `layoutWeight` | number | Row/Column 子组件弹性权重 |
| `flexShrink` | number | 0 表示不收缩，桌面卡片文本常设为 0 |
| `shadow` | object/string | 阴影对象或预设名 |
| `visibility` | string | `visible`、`hidden`、`none` |
| `clip` | boolean | 是否裁剪超出边界内容 |

## 尺寸与间距

`width` / `height` 可用：

| 写法 | 说明 |
|---|---|
| `160` | 160vp |
| `"160vp"` | 显式 vp |
| `"50%"` | 父容器百分比 |
| `"matchParent"` | 填充父容器内容区 |
| `"wrapContent"` | 随内容自适应 |
| `"fixAtIdealSize"` | 不受父组件内容区约束的理想尺寸 |

`margin` / `padding` 支持统一值或对象值：

```json
{"padding":{"top":12,"right":10,"bottom":8,"left":10}}
```

## 背景图与图片叠层

`backgroundImage` 可用于弱装饰底纹；如果图片是产品、设备、人物或场景识别主体，优先使用：

```json
{"id":"root","component":"Stack","children":["image","content"],"styles":{"width":"100%","height":160,"clip":true,"borderRadius":22}}
{"id":"image","component":"Image","src":{"path":"/asset/image"},"styles":{"width":"100%","height":"100%","objectFit":"cover"}}
```

这样可以独立控制 `Image.objectFit`、内容层避让、半透明承载底和点击区域。不要为了闭环改变 DSL 中资源路径；复制模板时保持图片与 DSL 的相对目录结构一致。

## linearGradient

推荐最小兼容写法：

```json
{"linearGradient":{"direction":"RightBottom","colors":[["#F2D89A",0],["#8EA2D8",0.5],["#5D78B8",1]]}}
```

规则：

- `direction` 可用 `Left`、`Right`、`Top`、`Bottom`、`LeftTop`、`LeftBottom`、`RightTop`、`RightBottom`、`None`。
- `colors` 推荐使用 `[颜色, stop]` 元组，stop 为 0-1。
- 桌面卡片生成不使用独立 `stops` 和 `repeating`，保持端侧兼容。

## shadow

对象写法：

```json
{"shadow":{"radius":16,"color":"#33000000","offsetX":0,"offsetY":4,"fill":false,"type":"color","style":"outer"}}
```

字段：

| 字段 | 必填 | 说明 |
|---|---:|---|
| `radius` | 是 | 模糊半径 |
| `color` | 否 | 阴影颜色 |
| `offsetX` / `offsetY` | 否 | 偏移 |
| `fill` | 否 | 是否内部填充，默认 false |
| `type` | 否 | `color` 或 `blur` |
| `style` | 否 | 对象内预设样式 |

可用预设名：`outerDefaultXS`、`outerDefaultSM`、`outerDefaultMD`、`outerDefaultLG`、`outerFloatingSM`、`outerFloatingMD`。

## visibility

| 值 | 布局行为 |
|---|---|
| `visible` | 可见 |
| `hidden` | 不可见但占位 |
| `none` | 不可见且不占位 |

隐藏辅助内容或小屏侧栏时优先用 `none`。

## 字段快照

下表综合组件单页 Reference 与聚合 Schema。若位置冲突，按 [source-decisions.md](source-decisions.md)：组件单页 Reference 的字段位置优先，聚合 Schema 用于补充简名组件字段和事件名。

| 结构 | 字段 |
|---|---|
| `CommonStyles` | `backgroundImageSizeWithStyle`、`flexShrink`、`width`、`height`、`constraintSize`、`backgroundImage`、`margin`、`borderRadius`、`visibility`、`clip`、`backgroundColor`、`borderWidth`、`borderColor`、`padding`、`layoutWeight`、`shadow`、`linearGradient` |
| `EventHandler` | `call`、`as`、`condition`、`args` |
| `TemplateChildren` | `componentId`、`path`、`indexVar`、`itemVar` |

组件字段快照：

| 组件 | 字段 |
|---|---|
| `Text` | `component`、`content`、`styles` |
| `Button` | `component`、`label`、`enabled`、`action`、`styles` |
| `TextInput` | `component`、`text`、`placeholder`、`enabled`、`maxLength`、`type`、`onChange`、`styles` |
| `Navigation` | `component`、`children`、`currentIndex`、`title`、`styles` |
| `Tabs` | `component`、`barPosition`、`children`、`vertical`、`scrollable`、`tabIndex`、`onChange` |
| `TabContent` | `component`、`title`、`icon`、`selectedSrc`、`tabType`、`styles` |
| `Select` | `component`、`options`、`selected`、`value`、`onSelect`、`styles` |
| `Web` | `component`、`url` |
| `Row` | `component`、`children`、`itemMargin`、`justifyContent`、`alignItems`、`wrap`、`styles` |
| `Column` | `component`、`children`、`itemMargin`、`justifyContent`、`alignItems`、`styles` |
| `List` | `component`、`children`、`space`、`listDirection`、`scrollBar`、`nestedScroll`、`onReachStart`、`onReachEnd`、`styles` |
| `Stack` | `component`、`children`、`alignContent`、`styles` |
| `Grid` | `component`、`children`、`columnsTemplate`、`rowsTemplate`、`columnsGap`、`rowsGap`、`styles` |
| `Image` | `component`、`src`、`styles` |
| `Divider` | `component`、`styles` |
| `Toggle` | `component`、`label`、`isOn`、`enabled`、`onChange`、`styles` |
| `Progress` | `component`、`value`、`total`、`styles` |
| `Radio` | `component`、`value`、`checked`、`group`、`onChange`、`styles` |
| `Checkbox` | `component`、`label`、`group`、`select`、`onChange`、`styles` |
| `CheckboxGroup` | `component`、`group`、`selectAll`、`onChange`、`styles` |
| `If` | `component`、`id`、`condition`、`childrenIf`、`childrenElse` |

字段参考若与本快照冲突，先对照当前外部开发文档修订本快照与字段参考；端侧渲染经验只能记录为兼容风险，不能覆盖开发文档。
