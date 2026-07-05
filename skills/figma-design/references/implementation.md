# Implementation — Coding Rules

用于在 Blueprint 和 Architecture 完成后编码。不得跳过前置门禁。

## Inputs

- Layer / State Coverage Matrix
- Dimension Spec / Box Model Check
- Icon & Asset Inventory
- Component Mapping
- DOM Audit Plan

## General Rules

- 先实现结构和状态机，再写样式细节。
- 使用项目既有组件和 tokens；没有明确需求不得新增 UI 库。
- CSS 覆盖只覆盖 Figma diff 项，不做无关重写。
- 保持 KISS / DRY / SSOT，避免重复状态。
- 业务 wrapper 承载业务逻辑，通用组件保持纯净。

## UI Library Override Rules

- 覆盖 UI library 样式前，先确认渲染后的真实 DOM 层级和 selector 是否命中。
- CSS Modules 的业务 class 不保证影响内部控件；需要 scoped internal selector 时，只覆盖 Figma diff 所需的最小节点。
- Figma 指定控件尺寸时，必须验证真实 input / selector / checkbox box，而不是只设置外层 wrapper。
- 如果 wrapper 与内部 DOM 尺寸不一致，先修内部真实控件，再复测 DOM Diff。

## State Machine First

多状态组件先列出状态：

| State | Trigger | Data | UI | Action |
|---|---|---|---|---|

必须覆盖 loading、empty、error、disabled、selected、hover/tooltip、submit pending、submit result。

## Modal Rules

- 先锁 modal root width/height。
- 再锁 header、body、footer 的 y/height。
- 禁止把 Modal 默认 body padding 当成 Figma padding。
- 使用 `modalRender`、className 或 wrapper 覆盖时必须可测量。

## List Rules

必须区分：

- viewport height
- scroll content height
- row height
- inner content inset
- avatar/text/action 子元素位置

列表为空时，empty state 的容器位置、插画、文案、按钮仍要按 Figma 单独测。

## Tabs Rules

- Tabs 必须受控，点击后 active state 和内容同步。
- tab label 文案、count、height、padding、active color、indicator 必须入 Dimension Spec。
- tabs 与 body、footer 的分割线必须按 Figma 单独实现或覆盖。

## Tooltip / Popover Rules

- `statusTips` / `info` / `question` icon 必须查 tooltip。
- tooltip 内容、触发方式、位置、宽度、背景、箭头、阴影必须实现或登记偏差。
- 不得只放 icon 不放 tooltip。

## Button Rules

按钮必须记录并实现：文案、icon、宽高、padding、radius、font、disabled/loading、左右对齐关系。

## Icon Rules

- 使用真实 SVG/Icon 组件，不用字符占位。
- Figma icon name → code icon 映射见 `icon-asset-rules.md`。
- 找不到 icon 时先搜索项目和 node_modules；仍找不到则记录偏差或导出资源。

## Error Handling

错误展示应按业务契约和项目规范映射。不得将 transport status（如 HTTP 409/502）硬编码成业务状态，除非 spec 明确要求。

## Output Gate

编码结束不代表完成。必须进入 `verify.md` 做 DOM Diff。
