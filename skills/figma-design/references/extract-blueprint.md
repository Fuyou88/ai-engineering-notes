# Extract Blueprint — Measurement, Icons, Assets

用于从 Figma 提取可编码 Blueprint。绝对尺寸必须来自 `get_metadata`。

## Inputs

- Intake 输出的 node list / scope
- Figma API budget，默认 10
- 目标框架：React / TypeScript / CSS / Less

## Figma MCP Call Order

对每个关键 node 串行调用，禁止并发：

1. `get_design_context(nodeId, clientFrameworks="react", clientLanguages="typescript,css")`
2. `get_metadata(nodeId)` ← 尺寸唯一可信来源
3. `get_screenshot(nodeId)` ← 视觉基线

每次调用前检查 budget；同一 nodeId 同一会话走缓存。

## Figma Payload Hygiene

不要把大型 Figma raw payload 直接贴进对话或最终响应。

对于大型 metadata / screenshot / design_context：

- 优先将 raw payload 保存在本地临时文件或工具结果中。
- 只把 compact structured artifacts 放入上下文：Node Index、Layer Coverage Matrix、State Coverage Matrix、Dimension Spec。
- 不在响应中包含 screenshot base64。
- 截图使用文件路径或视觉工具引用。

## Two-pass Extract

### Pass 1 — Inventory

用于 root/page/section nodes：

- 只获取 root metadata 一次。
- 提取紧凑节点清单：id、name、type、parent、x/y/w/h、hidden、coarse scenario classification。
- 先构建 Layer Coverage Matrix 和 State Coverage Matrix。
- 不在第一遍分析所有 leaf 细节。

### Pass 2 — Focused Blueprint

只对 in-scope implementation nodes 展开：

- 提取详细尺寸和盒模型。
- 只展开关键 descendants：layout containers、rows、buttons、icons、empty states、tooltip/popover、assets。
- 若缺少实现阻塞细节，再打开 raw metadata。

## Summary-first Cache

每个已获取 Figma node 保留：

- raw payload location（如可用）
- compact summary
- key dimension table
- screenshot path（如可用）

复访 node 时：

1. 先使用 compact summary。
2. 只有缺失细节阻塞实现或验证时，再打开 raw metadata。

## Design Context Strategy

选择性使用 `get_design_context`。

Recommended:

- component-level nodes，需要语义样式或 generated code。
- 不熟悉的组件库或 design tokens。
- metadata 名称不足以判断 icon/asset 含义时。

Avoid:

- 对每个 state frame 都调用 design_context。
- 对巨大 root section 调用，除非确实需要语义信息。

若 design_context 返回 Code Connect prompt 或无关大段代码：标记 degraded，继续使用 metadata + screenshot。

## Screenshot Strategy

截图只作为 visual baseline，不作为文字上下文。

- 不把 screenshot base64 贴进 prompt/response。
- 保存截图到文件或使用视觉工具。
- 大型 root screenshot 只用于 overview。
- 像素验证优先截图关键组件节点，不截整页替代 DOM measurement。
- Screenshot comparison 是补充；完成前仍必须做 DOM measurement。

## Degraded Mode Matrix

| design_context | metadata | screenshot | Decision |
|---|---|---|---|
| ✅ | ✅ | ✅ | 完整 Blueprint |
| ✅ | ✅ | ❌ | 可编码，截图缺失记录为偏差 |
| ✅ | ❌ | any | 仅语义，不得做像素级完成声明 |
| ❌ | ✅ | any | 仅尺寸，可继续 Icon/CSS 手动审计 |
| ❌ | ❌ | ✅ | 仅视觉，建议用户补 MCP/metadata |
| ❌ | ❌ | ❌ | 停止 Extract |

若 `design_context` 被 Code Connect prompt 阻断，继续使用 metadata + screenshot，但在 Blueprint 中标注。

## Metadata Unavailable Fallback

若当前 Figma MCP 不提供 `get_metadata`，不得假装已有精确 px metadata：

1. 优先从 `get_design_context` 提取可见的 layout hints（如 generated code 中的 `w/h/gap/padding/px/py/size`）。
2. 将这些值标注为 `⚠️ approximate from design_context`，只可作为实现参考，不可作为 pixel-perfect 完成依据。
3. 必须保留 Browser DOM measurement；最终验证以 DOM Diff / Adjacent Box Boundary Diff 是否满足接受阈值为准。
4. 若缺少 metadata 导致无法判断关键尺寸，停止 Extract 并要求用户补充 Figma MCP metadata 或提供更明确节点截图。

## Dimension Spec Template

| Node | Element | Parent | x | y | width | height | CSS Target | Notes |
|---|---|---|---:|---:|---:|---:|---|---|

## Gap Calculation

```text
gap = nextChild.y - (currentChild.y + currentChild.height)
```

横向同理：

```text
gapX = nextChild.x - (currentChild.x + currentChild.width)
```

## Padding Calculation

```text
padding-top = firstChild.y - parent.y
padding-left = firstChild.x - parent.x
padding-bottom = parent.height - ((lastChild.y - parent.y) + lastChild.height)
padding-right = parent.width - ((rightmostChild.x - parent.x) + rightmostChild.width)
```

## Box Model Check

每个 container 必须做盒模型加法：

```text
padding-top + Σ(child.height) + Σ(vertical gap) + padding-bottom = container.height
padding-left + Σ(child.width) + Σ(horizontal gap) + padding-right = container.width
```

| Container | Equation | Expected | Actual Sum | Result |
|---|---|---:|---:|---|

## Typography Spec

Figma `line-height: normal` 不可信。包含文本的行容器必须用 metadata height 反推显式 line-height / row height。

| Node | Text | Figma height | Font size | Figma line height | Code decision | Notes |
|---|---|---:|---:|---|---|---|

## Icon Inventory

扫描 icon 节点，不得用文本字符占位。

| Figma Name | Node | Size | Code Icon | Package | Exists? | Usage |
|---|---|---:|---|---|---|---|

本项目常见映射见 `icon-asset-rules.md`。

## Asset Inventory

非 icon 图片、空态插画、背景纹理必须登记并做资源策略。

| Node | Asset Name | Type | Size | Strategy | Dark Mode? | Notes |
|---|---|---|---:|---|---|---|

策略只能是：existing asset / export asset / CSS recreate with explicit acceptance / defer with deviation。

## CSS Override List

仅对第三方组件默认样式与 Figma 差异项覆盖。

| Component | Default CSS | Figma | Override | Reason |
|---|---|---|---|---|

## Output Gate

编码前必须已有：

- Dimension Spec
- Box Model Check
- Typography Spec
- Icon Inventory
- Asset Inventory
- CSS Override List
