# DSL 生成指导规范

> **用途：** 选定模板后（见 [template.md](template.md)），按本指南执行 DSL 生成、自检和校验。本指南负责生成流程与自检；协议细节见 [protocol-core.md](protocol-core.md)，组件字段见 [component-field-reference.md](component-field-reference.md)。

## 前置条件

生成 DSL 前必须已完成：
1. capability.md 能力判断 — 确认需求可满足
2. template.md 尺寸推断/模板选择 — 用户指定尺寸则直接匹配；未指定则根据 query 信息量、操作数量、排版结构和组件估算选择最小可承载尺寸
3. **渐变色彩选择（选用渐变时）** — 从 template.md 语义色板速查表和三段渐变模型匹配用户 query 语义
4. **图片叠层判断（选用背景图时）** — 图片必须承载产品/设备/场景识别；若只是装饰，优先改用语义渐变
5. **整体结构排版确认** — 在写任何具体组件字段前，必须先确定卡片尺寸、根容器、信息槽位、主次层级、组件邻接表和每个槽位的大致尺寸/间距。

若跳过了以上步骤，先返回对应文件完成判断。

---

## 一、消息输出顺序（强制）

每条 JSONL 必须按以下顺序输出，不可调换：

```
第 1 行：createSurface        ← 创建渲染 Surface
第 2 行：updateComponents     ← 提交完整组件骨架（可多条）
第 3 行：updateDataModel      ← 初始化数据（path: "/"）
```

额外规则：
- `surfaceId` 在所有消息中必须一致。
- `updateComponents` 可拆分为多条消息（增量更新），但初始化时建议一条包含全部组件。
- `updateDataModel` 后续可发送增量更新（如 `path: "/weather"`），只更新子树。
- 不要在 JSONL 中包含解释文字、Markdown、注释、尾随逗号或跨行不完整对象。

---

## 二、createSurface 规范

```json
{"version":"v0.9","createSurface":{"surfaceId":"<kebab-case-id>","catalogId":"ohos.a2ui.extended.catalog","theme":{"primaryColor":"#RRGGBB"}}}
```

| 字段 | 必填 | 约束 |
|------|------|------|
| `surfaceId` | 是 | 短横线命名，稳定且有语义，如 `"parents-care-card"` |
| `catalogId` | 是 | 固定 `"ohos.a2ui.extended.catalog"` |
| `theme.primaryColor` | 建议 | 品牌主色，影响按钮、进度条、强调文字 |

---

## 三、updateComponents 规范

### 3.0 先整体结构，后组件细节（强制）

创建组件时必须分两轮完成，禁止一边想布局一边逐个堆组件字段：

**第一轮：确定整体结构排版。**
1. 选定卡片规格和布局骨架，例如 `root → heroRow/contextCard/actionPill`、`root → topRow/bottomRow`、`root(Stack) → image/contentLayer`。
2. 把 `primary`、`context`、`details`、`action`、`state` 放入明确槽位，并确认每个槽位只承担一种信息角色。
3. 写出扁平组件邻接表：所有容器只引用已规划的子组件 ID；先确认入口 `root`、层级深度、同级顺序、是否需要 `If` 分支或模板列表。
4. 预估每个槽位的宽高、padding、itemMargin、可见文本行数和操作数量，确保不超过所选尺寸上限。

**第二轮：填写组件内部细节。**
1. 先为容器填写布局字段：`children`、`itemMargin`、`justifyContent`、`alignItems`、必要的 `width`/`height`/`padding`。
2. 再为叶子组件填写展示字段：`Text.content`、`Image.src`、`Button.label`、`Progress.value/total`。
3. 最后补视觉和交互：`fontSize`、`fontColor`、`backgroundColor`、`linearGradient`、`borderRadius`、`onClick`、`accessibility`。
4. 最后设计 DataModel，保证每个 `{ "path": "/..." }` 都有初始值。

若渲染结果出现裁剪、错位或层级混乱，先回到第一轮重排结构，不要只微调单个组件的字号、padding 或颜色。

### 3.1 组件描述符通用结构

```json
{
  "id": "<unique-id>",
  "component": "<ComponentName>",
  "styles": { /* 通用样式 + 组件专有样式 */ },
  "accessibility": { "label": "...", "description": "..." },
  "onClick": [ /* EventHandler[] */ ],
  "onAppear": [ /* EventHandler[] */ ]
}
```

### 3.2 字段层级兼容矩阵

字段归属是最常见错误来源。当前项目中，单组件 Reference 和聚合 Schema 对部分布局字段存在顶层/`styles` 差异；按 [source-decisions.md](source-decisions.md) 的优先级，组件字段位置优先遵循组件单页 Reference，并保持同一 DSL 内只写一种位置，禁止同一字段双写。

| 字段 | 所属层级 | 说明 |
|------|---------|------|
| `id`、`component` | 顶层 | 所有组件通用 |
| `children`、`child` | 顶层 | 布局容器 |
| `content` | 顶层 | Text 组件 |
| `src` | 顶层 | Image 组件 |
| `label` | 顶层 | Button 组件 |
| `value`、`total` | 顶层 | Progress 组件 |
| `condition`、`childrenIf`、`childrenElse` | 顶层 | If 组件 |
| `itemMargin` | 顶层 | Column/Row 子项间距 |
| `justifyContent`、`alignItems` | 顶层 | Column/Row 对齐；组件单页 Reference 写法 |
| `space` | 顶层 | List 子项间距 |
| `scrollBar`、`listDirection`、`nestedScroll` | 顶层 | List 方向与滚动；组件单页 Reference 写法 |
| `columnsTemplate`、`rowsTemplate`、`columnsGap`、`rowsGap` | 顶层 | Grid 布局；组件单页 Reference 写法；不要使用 `repeat()` |
| `width`、`height`、`constraintSize` | `styles` 内 | 尺寸 |
| `margin`、`padding` | `styles` 内 | 内外边距 |
| `backgroundColor`、`backgroundImage`、`linearGradient` | `styles` 内 | 背景 |
| `borderRadius`、`borderWidth`、`borderColor` | `styles` 内 | 边框圆角 |
| `shadow` | `styles` 内 | 阴影 |
| `fontSize`、`fontWeight`、`fontColor` | `styles` 内 | 文本样式 |
| `maxLines`、`minFontSize`、`maxFontSize` | `styles` 内 | Text 组件 |
| `textOverflow`、`textAlign`、`wordBreak` | `styles` 内 | Text 组件 |
| `objectFit`、`aspectRatio` | `styles` 内 | Image 组件 |
| `color`、`type` | `styles` 内 | Progress/Divider 组件 |
| `vertical`、`strokeWidth` | `styles` 内 | Divider 组件 |
| `visibility`、`clip` | `styles` 内 | 显隐裁剪 |
| `layoutWeight`、`flexShrink` | `styles` 内 | 弹性布局 |

**常见放错位置（必须避免）：**
- ❌ 顶层和 `styles` 同时设置同一布局字段 → ✅ 只保留一种位置
- ❌ `"styles":{"itemMargin":12}` → ✅ 顶层 `"itemMargin":12`
- ❌ `"fontSize":"16fp"` → ✅ `"fontSize":16`（纯数字）
- ❌ `{"component":"Text","text":"..."}` → ✅ `"content":"..."`
- ❌ `{"component":"Image","url":"..."}` → ✅ `"src":"..."`

**关键 `styles` 字段补充说明：**

`visibility` 枚举：`"visible"`（可见）、`"hidden"`（不可见但占位）、`"none"`（不可见且不占位）。隐藏辅助内容用 `"none"`。

`shadow` 预设（可使用字符串快捷方式替代完整对象）：

| 预设名 | 效果 |
|--------|------|
| `outerDefaultXS` / `outerDefaultSM` / `outerDefaultMD` / `outerDefaultLG` | 默认外阴影（4 级深度） |
| `outerFloatingSM` / `outerFloatingMD` | 悬浮外阴影 |

也可用对象形式：`{"radius":8,"color":"#33000000","offsetX":0,"offsetY":2,"fill":false,"type":"color","style":"outer"}`。对象形式 `radius` 必填。

`linearGradient` 结构：`direction`（`Left`/`Right`/`Top`/`Bottom`/`LeftTop`/`LeftBottom`/`RightTop`/`RightBottom`/`None`）、`colors`（`[["#色1", 0], ["#色2", 0.5], ["#色3", 1]]`，每项为 `[颜色字符串, 色标位置]` 元组，色标直接嵌入颜色项）。当前聚合 Schema 声明了 `repeating`，但桌面卡片生成仍优先省略 `repeating` 和独立 `stops`，保持最小兼容写法。

### 3.3 子组件引用

**静态子组件（所有布局容器）：**
```json
{"children":["header","body","footer"]}
```

**按钮：**
扩展 `Button` 使用 `label` 字符串，不使用 `child`。如果目标渲染器覆盖 `Button.styles.fontColor`，需要精确控制 CTA 字色时使用可点击 `Row` + 内部 `Text`。

**模板列表（List、Grid 等容器）：**
```json
{
  "children": {
    "componentId": "itemTemplate",
    "path": "/data/items",
    "itemVar": "$item",
    "indexVar": "$index"
  }
}
```

### 3.4 三种动态值类型

| 类型 | 写法 | 使用场景 |
|------|------|---------|
| 字面量 | `"content":"上海"` | 静态文案 |
| 路径绑定 | `"content":{"path":"/weather/temperatureLabel"}` | 直接读取 DataModel |
| 表达式 | `"content":"{{ $__DataModel.weather.city + ' · ' + $__DataModel.weather.condition }}"` | 短文本拼接、条件文案、条件颜色 |

**优先级：路径绑定 > 表达式 > 字面量。** 能直接绑定的不要用表达式。

---

## 四、updateDataModel 规范

### 4.1 初始化数据

```json
{"version":"v0.9","updateDataModel":{"surfaceId":"example","path":"/","value":{ /* 完整数据树 */ }}}
```

### 4.2 DataModel 设计要求

1. **语义路径**：使用 `/weather/temperatureLabel` 而非 `/w/tmp`。
2. **展示值与原始值分离**：`temperatureC: 28` + `temperatureLabel: "28°"`。
3. **颜色放入数据**：`coldIndexColor: "#34A853"`，由数据驱动 UI 配色。
4. **预格式化展示字符串**：`weatherSummary: "上海 · 多云转晴  ｜  体感 26°"`。
5. **数组用于重复内容**：`items: [{...}, {...}]`。
6. **初始化每个绑定路径**：所有 `{ "path": "/..." }` 必须在初始 DataModel 中有值。
7. **示例数据必须虚构**：手机号用 `13800138000`，人名用代称，不暴露真实隐私。

### 4.3 增量更新

后续刷新时只发送变化的子树：
```json
{"version":"v0.9","updateDataModel":{"surfaceId":"example","path":"/weather","value":{"temperatureLabel":"29°","condition":"晴"}}}
```

---

## 五、表达式规范

### 5.1 适用场景

表达式仅在鸿蒙扩展组件目录 (`ohos.a2ui.extended.catalog`) 下可用：

```
✅ 短文本拼接："{{ $__DataModel.tasks.count + ' 项待办' }}"
✅ 条件文案："{{ $__DataModel.weather.coldIndex > 3 ? '注意防护' : '适宜外出' }}"
✅ 条件颜色："{{ $__DataModel.weather.coldIndex <= 2 ? '#34A853' : '#EA4335' }}"
✅ 断点字号："{{ $__WindowWidthBreakpoint == 'xs' ? 14 : 18 }}"
✅ 条件显隐："{{ $__DataModel.hasWarning ? 'visible' : 'none' }}"
✅ 模板循环："{{ $item.done ? '✓' : '○' }}"
```

### 5.2 禁止场景

```
❌ id 或 component 字段中使用
❌ { "path": "..." } 绑定内部使用
❌ 复杂业务逻辑（多层级嵌套三元）
❌ 假设支持 JavaScript 方法或任意代码执行
❌ 非标准断点名称（如 'medium'、'small'）
```

### 5.3 可用变量

| 变量 | 说明 | 示例值 |
|------|------|--------|
| `$__DataModel` | 完整数据模型 | `$__DataModel.weather.temperatureLabel` |
| `$__WindowWidthBreakpoint` | 窗口断点 | `xs`、`sm`、`md`、`lg`、`xl` |
| `$__ColorMode` | 色彩模式 | `light`、`dark` |
| `$item` | 模板当前项 | `$item.title` |
| `$index` | 模板当前索引 | `$index`（从 0 开始） |

---

## 六、交互事件规范

### 6.1 事件结构

事件字段值是 EventHandler 数组，不是单个字符串：

```json
{
  "onClick": [
    {
      "call": "makePhoneCall",
      "args": {"phoneNumber":"13800138000"}
    }
  ]
}
```

### 6.2 EventHandler 字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `call` | string | 是 | 调用函数名。无函数目录信息时使用**明确的占位契约名** |
| `args` | object | 否 | 调用参数，可使用 `"{{ $item.id }}"` 表达式 |
| `condition` | string | 否 | 执行条件表达式 |
| `as` | string | 否 | 保存返回结果的变量名 |

### 6.3 事件使用约束

- 无已注册函数目录信息时，`call` 使用语义明确的占位契约名（如 `"makePhoneCall"`），不要宣称它已由系统内置实现。
- 不得虚构端侧函数名或假设系统 API 可用。
- `onClick` 和 `onAppear` 属于协议通用事件，其余事件（如 `onChange`、`onSelect`、`onReachStart`、`onReachEnd`）仅在对应组件上使用。

---

## 七、多设备自适应规范

### 7.1 尺寸单位（仅扩展目录可用）

| 单位 | 说明 | 示例 |
|------|------|------|
| 纯数字 | 默认 vp（虚拟像素） | `16`（等同 `16vp`） |
| `"Nvp"` | 显式虚拟像素 | `"100vp"` |
| `"N%"` | 相对于父容器百分比 | `"100%"` |
| `matchParent` | 填充父容器 | `"width":"matchParent"` |
| `wrapContent` | 内容自适应 | `"height":"wrapContent"` |

**注意：`fontSize` 使用纯数字，不要写成 `"16fp"` 字符串。**

### 7.2 何时用 If 组件

当不同断点**布局结构不同**时（单列 vs 双列），使用 If：

```json
{
  "id": "adaptiveLayout",
  "component": "If",
  "condition": "{{ $__WindowWidthBreakpoint == 'xs' || $__WindowWidthBreakpoint == 'sm' }}",
  "childrenIf": ["narrowLayout"],
  "childrenElse": ["wideLayout"]
}
```

**If 铁律：**
- 必须同时提供 `childrenIf` 和 `childrenElse`。
- 两个分支引用的组件 ID 必须在同一条 `updateComponents` 中定义。
- 断点变量名是 `$__WindowWidthBreakpoint`（双下划线），不要写错。

### 7.3 何时用三元表达式

当不同断点只有**属性值不同**时（字号、间距）：

```json
{
  "fontSize": "{{ $__WindowWidthBreakpoint == 'xs' ? 14 : $__WindowWidthBreakpoint == 'sm' ? 16 : 20 }}"
}
```

### 7.4 断点分组建议

| 分组 | 断点 | 布局特征 |
|------|------|---------|
| 小屏 | xs + sm | 单列、紧凑间距、小字号 |
| 中屏 | md + lg | 双列可选、中等间距 |
| 大屏 | xl | 多列、大间距 |

---

## 八、生成后自检清单

DSL 写完后逐项核对（在运行校验脚本前）：

### 结构完整性
- [ ] 第一条消息是 `createSurface`
- [ ] 所有消息 `surfaceId` 一致
- [ ] 每行是一个完整 JSON 对象，无尾随逗号
- [ ] 无 Markdown 代码围栏、注释、解释文字
- [ ] 存在 `root` 组件且为唯一入口
- [ ] 已先确定整体布局骨架和组件邻接表，再填写组件内部字段；不存在边写字段边临时拼布局的痕迹
- [ ] 每个顶层槽位的信息角色明确：`primary`、`context`、`details`、`action` 或 `state`，没有多个角色挤在同一小组件中

### 组件引用
- [ ] 所有 `children`/`child`/`childrenIf`/`childrenElse` 引用的组件 id 均已定义
- [ ] 无循环引用或自引用
- [ ] 模板组件 (`componentId`) 与容器在同一组件集合中
- [ ] 无内联组件对象（children 数组只含字符串 id）

### 字段正确性
- [ ] `id` 和 `component` 无动态绑定
- [ ] Text 使用 `content` 而非 `text`
- [ ] Image 使用 `src` 而非 `url`
- [ ] Button 使用 `label` 而非 `child`
- [ ] Row 使用 `justifyContent`/`alignItems` 而非 `justify`/`align`
- [ ] `fontSize` 为纯数字，非字符串
- [ ] `itemMargin` 在顶层；`justifyContent`/`alignItems` 若顶层或 `styles` 出现，只出现一次
- [ ] List/Grid 的方向/网格字段没有顶层与 `styles` 双写
- [ ] 视觉属性 (`backgroundColor`、`padding`、`fontColor` 等) 在 `styles` 内
- [ ] 颜色格式 `#RRGGBB` 或 `#AARRGGBB`
- [ ] Grid `columnsTemplate` 不使用 `repeat(...)` 语法

### 视觉对比与边界
- [ ] 文字与所在背景有明显明暗差：深底用白色透明阶，浅底用黑色透明阶，不使用与背景同色相同亮度的文字
- [ ] CTA 的文字色与按钮底色形成清晰对比；半透明深色/彩色按钮优先白字，浅色按钮使用深字
- [ ] 若使用 `Button.styles.fontColor`，已考虑端侧主题色覆盖风险；要求精确字色时改用可点击 `Row` + `Text`
- [ ] 内层信息卡、状态块、按钮与背景之间至少有一种清晰边界：明度差、1vp 描边、阴影或足够留白
- [ ] 文字颜色、区域颜色与背景颜色三层清晰分离，不存在“单层可读但叠加后融在一起”的情况
- [ ] CTA 同时通过两层检查：内对比（文字 vs 按钮底色）和外对比（按钮轮廓 vs 按钮所在局部背景）
- [ ] 任一文字、信息区或 CTA 若位于特殊局部背景（深色、浅色、高饱和、低亮度、高亮度收束区等），未继续使用同色相近明度的文字色、区域色或描边
- [ ] 同级子卡片的背景透明度、描边、圆角、padding 保持一致，主操作入口可更突出但不混淆层级
- [ ] 模块间距大于模块内部行距，避免多个组件贴在一起导致边界不清
- [ ] 使用真实图片时，图片主体没有被文字、状态卡或 CTA 遮挡；文字层根据图片明暗使用独立承载底或遮罩

### DataModel
- [ ] 所有 `{ "path": "/..." }` 绑定的路径在初始 DataModel 中有值
- [ ] 路径使用语义化命名
- [ ] 数组字段使用模板列表而非固定数量组件
- [ ] 示例数据为虚构值，不暴露真实隐私

### 卡片尺寸
- [ ] 用户显式指定尺寸时，已严格使用该尺寸；用户未指定时，已按 query 信息槽位、操作数量、排版结构和组件估算选择最小可承载尺寸
- [ ] 未指定尺寸时没有默认套用 2×2；若选择 2×2，是因为内容结构和硬限匹配 2×2
- [ ] 组件数量、文本行数、按钮数在所选尺寸的硬限内
- [ ] 字体大小在所选尺寸的推荐范围内
- [ ] `root` 宽度为 `"100%"` 或 `"matchParent"`
- [ ] 所有文本使用 `maxLines` + `textOverflow` 防护
- [ ] 间距（padding、itemMargin）在所选尺寸的推荐范围内
- [ ] 每张卡片只有 1 个主任务或主指标，辅助信息没有抢占主视觉焦点
- [ ] 布局与信息结构匹配：主任务分层卡用于状态/上下文/操作，上下分区行动卡用于顶部状态+底部详情/动作，图像叠层卡用于真实主视觉，象限网格用于多槽位信息，单列流式用于同类列表

### 2×2 象限网格专项（使用该布局时必检）
- [ ] 上下两个 Row 均设置 `"layoutWeight":1`（值相同，确保行高等分）
- [ ] 每行内左右 Column 均设置 `"layoutWeight":1` + `"height":"matchParent"`
- [ ] root Column 设置 `"itemMargin":0`（不留行间缝隙）
- [ ] 左右 Column 的 `padding.top` 和 `padding.bottom` 对应相等
- [ ] 子卡片（如指标卡）使用固定数值 `width`/`height`，不使用 `"matchParent"` 或百分比
- [ ] 所有 Text 设置 `"flexShrink":0`（防止弹性收缩）
- [ ] 渐变使用 `"direction":"RightBottom"`，色标 2-3 个，配合 `"clip":true`
- [ ] **渐变色彩从语义色板速查表匹配用户 query 语义**，非固定色值
- [ ] 辅助文字色跟随选定场景的推荐值（暖色系/冷色系各有对应辅助文字色）
- [ ] 渐变符合三段模型：受光色、低/中饱和主题色、深色收束；避免首尾都高饱和或尾色近黑
- [ ] 象限对齐符合约定：左上 start/start，右上 start/end，左下 end/start，右下 end/end

### 2×2 主任务分层卡专项（状态/提醒/控制/行动类必检）
- [ ] 结构为 `heroRow + contextCard + actionPill` 三层或等价三层结构，主信息、上下文、操作入口不混在同一层
- [ ] 顶区只有 1 个主视觉焦点，其余文本字号明显降低，避免三段内容等权
- [ ] `contextCard` 使用半透明背景 + 1vp 浅描边 + 14-18vp 圆角，形成内嵌浮层
- [ ] CTA 优先使用同色系半透明 pill，并通过白字/深字、描边或明度差保证可点击边界清晰；除超深色卡片外，不使用纯白大按钮 + 高饱和品牌色文字
- [ ] 当 CTA 或信息区压在复杂局部背景上，优先改用更大明度差、更清晰描边、独立承载底或更中性的区域色方案，而不是沿用主题色的近明度变体
- [ ] 全卡图标锚点控制在 1-2 个，不给每行文本都加 emoji 前缀

### 2×2 上下分区行动卡专项（顶部状态 + 底部动作必检）
- [ ] 结构为 `topRow + bottomRow` 或等价上下两段，根容器优先 `height:160`、`justifyContent:"spaceBetween"`
- [ ] 顶部只突出一个大指标或视觉状态，另一个元素作为辅助；大指标和大图标不同时使用强强调色
- [ ] 底部详情卡使用固定尺寸和半透明承载底，圆形操作按钮使用固定宽高且 `borderRadius` 为半宽
- [ ] 底部详情卡与圆形按钮都与局部背景有清晰边界，不能只靠同色系透明度区分

### 2×2 图像背景叠层卡专项（产品/设备/场景必检）
- [ ] 根组件使用 `Stack`，第一层为全尺寸 `Image`，第二层为全尺寸内容层；根容器设置 `height:160`、`borderRadius`、`clip:true`
- [ ] `Image` 使用 `src` 路径或资源绑定，`width:"100%"`、`height:"100%"`、`objectFit:"cover"`；不要把主要识别图片放进 `backgroundImage`
- [ ] 文本和状态块避开图片主体；浅图用深字 + 半透明白卡，深图用白字 + 暗色遮罩或半透明深卡
- [ ] 组件数若超过 15，必须属于图片叠层/设备状态类，且总数不超过 22、可见文本不超过 6 行、主要可点击入口不超过 1 个

---

## 九、校验与修复

### 运行校验

在 `generate-harmonyos-a2ui-widget` 技能根目录下运行：

```powershell
python scripts/validate_a2ui_jsonl.py <JSONL文件路径>
```

### 修复优先级

1. **错误（必须修复）**：协议版本错误、组件目录错误、JSON 格式错误、缺少 root、组件 id 重复、悬空引用、文字层/区域层/背景层区分失败、CTA 内对比失败、CTA 外轮廓融入局部背景（例如亮蓝 CTA 贴深蓝尾部渐变）
2. **警告（设计复核）**：字段放错层级、fontSize 格式、模板未使用、Grid `repeat()` 语法

### 常见误写修复对照

审查已有 DSL 时按以下规则修复：

| 错误写法 | 正确写法 | 说明 |
|---------|---------|------|
| `Text.text` | `Text.content` | 扩展组件属性名 |
| `Image.url` | `Image.src` | 扩展组件属性名 |
| `Button.child` | `Button.label` | 扩展 Button 用 label |
| `Row.justify` | `Row.justifyContent` | 扩展组件属性名 |
| `Row.align` | `Row.alignItems` | 扩展组件属性名 |
| `Extended.Button` / `Extended.Text` | `Button` / `Text` | 去 `Extended.` 前缀 |
| `Extended.Select` / `Extended.Tabs` | `Select` / `Tabs` | 新生成优先简名，旧写法仅兼容 |
| `Select.onChange` | `Select.onSelect` | 简名 `Select` 按聚合 Schema 使用 `onSelect` |
| `"fontSize":"16fp"` | `"fontSize":16` | 纯数字，非字符串 |
| `columnsTemplate:"repeat(2, 1fr)"` | `columnsTemplate:"1fr 1fr"` | 不支持 repeat() |
| `If` 只有 `childrenIf` | 补充 `childrenElse` | If 必须双分支 |
| `linearGradient` 使用 `stops` + `repeating` | `colors` 用 `[["#色",0],["#色",0.5],...]` 元组 | 色标嵌入 colors，无 stops/repeating 字段 |
| 视觉属性在组件顶层 | 移入 `styles` | 如 backgroundColor 等 |
| 同一布局属性顶层和 `styles` 双写 | 保留一种 | 兼容不同文档版本，但不要重复 |

### 修复时禁止的操作

- 不要为了让错误 DSL 通过而静默切换组件目录
- 不要将遗漏的 `childrenElse` 留空
- 不要将不存在的组件前缀改写为错误的名称
- 不要删除校验失败的组件而不修复引用

---

## 十、完整最小示例

```jsonl
{"version":"v0.9","createSurface":{"surfaceId":"mini-metric","catalogId":"ohos.a2ui.extended.catalog","theme":{"primaryColor":"#0A59F7"}}}
{"version":"v0.9","updateComponents":{"surfaceId":"mini-metric","components":[{"id":"root","component":"Column","children":["metricLabel","metricValue","metricSummary"],"itemMargin":4,"alignItems":"center","styles":{"width":"100%","padding":12}},{"id":"metricLabel","component":"Text","content":{"path":"/metric/label"},"styles":{"fontSize":12,"fontColor":"#99000000","maxLines":1,"textOverflow":"ellipsis"}},{"id":"metricValue","component":"Text","content":{"path":"/metric/valueLabel"},"accessibility":{"label":"主指标"},"styles":{"fontSize":36,"fontWeight":"bold","fontColor":"#E6000000","maxLines":1}},{"id":"metricSummary","component":"Text","content":{"path":"/metric/summary"},"styles":{"fontSize":13,"fontColor":"#CC000000","maxLines":1,"textOverflow":"ellipsis"}}]}}
{"version":"v0.9","updateDataModel":{"surfaceId":"mini-metric","path":"/","value":{"metric":{"value":128,"valueLabel":"128","label":"当前指标","summary":"运行正常"}}}}
```

---

## 相关文档

- 参考入口：[README.md](README.md)
- 卡片能力范围：[capability.md](capability.md)
- 协议核心：[protocol-core.md](protocol-core.md)
- 通用字段与样式：[common-fields-and-styles.md](common-fields-and-styles.md)
- 卡片尺寸模板：[template.md](template.md)
- 扩展组件索引：[extended-component-catalog.md](extended-component-catalog.md)
- 扩展组件字段参考：[component-field-reference.md](component-field-reference.md)
- 桌面卡片模板参考：[dsl-templates/](dsl-templates/)
- 开发文档引入决策：[source-decisions.md](source-decisions.md)
