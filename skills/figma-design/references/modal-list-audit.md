# Modal/List/Tabs/Button/Avatar Audit

用于所有弹窗、列表、Tabs、Footer、Avatar 密集 UI 的专项审计。本文件沉淀本次投票名单弹窗的教训：第三方组件外层、滚动容器、内部列表常有多层盒模型，不能凭截图或 CSS 猜。

## Extract Separately

Standard / Full 的 modal/list 设计必须分别提取；Simple 修复只提取与目标偏差相关的 row / footer / child sequence：

1. Modal root
2. Header
3. Tabs / toolbar
4. Content viewport
5. Scrollbar wrapper
6. Scroll content
7. Row
8. Avatar
9. Primary text / secondary text
10. Empty state
11. Footer
12. Buttons
13. Divider lines

## Distinguish These Boxes

| Concept | Meaning | Common Bug |
|---|---|---|
| Modal root | 整个弹窗外框 | 只设 width，不设 height |
| Body viewport | 可视内容区 | 被第三方 padding 挤压 |
| Scrollbar wrapper | UI library 滚动容器 | 误当 list root |
| Scroll content | 实际滚动内容高度 | 与 viewport 混淆 |
| List inset | 列表内边距 | 误用外 margin |
| Row | 每行高度 | 行高和 gap 重复计算 |
| Footer | 底部操作区 | 分割线、padding、按钮对齐偏差 |

## Third-party DOM Audit

必须测量或审计：

- `.modal-content`
- `.modal-header`
- `.modal-body`
- `.scrollbar`
- `.scrollbar-view`
- tabs root / nav / ink-bar
- button root / icon / text

## Wrapper Spacing Risk

当子元素是第三方组件或包装原生控件时，必须测量真实参与布局的 wrapper box。常见隐藏间距来源包括 `margin`、`padding`、inline wrapper 宽度、label/input 组合；不要只测内部 icon/img。

示例：Figma row gap 为 `checkbox.right → avatar.left = 8`，DOM 实测为 `28`；根因是 checkbox wrapper `margin-right: 20px` 与 row gap `8px` 叠加。

## DOM Measurement Targets

| Target | Required Measurement |
|---|---|
| modal rect | x/y/width/height |
| header rect | y/height/padding |
| tabs rect | y/height/indicator |
| body rect | y/height/padding/overflow |
| list wrapper | x/y/width/height/margin/padding |
| first row | x/y/width/height |
| first avatar | x/y/width/height |
| text block | x/y/width/height/line-height |
| empty state | center position/asset/text |
| footer rect | y/height/border/padding |
| buttons | x/y/width/height/gap/text |

## Footer Rules

- footer top divider 是独立元素或 border，必须与 Figma y 坐标一致。
- 按钮文案必须逐字匹配 Figma。
- 左右按钮位置按 x/width 计算，不用 `space-between` 盲猜。

## Empty State Rules

- 空态不是简单文本居中；需要提取插画、容器、文案、按钮位置。
- 若没有资源导出，必须记录为 Asset Deviation，不能用 CSS 图形假装 1:1。

## Output Gate

涉及 modal/list 时，DOM Diff Report 必须至少包含 modal、body、tabs、list wrapper、row、footer、button 的 diff。
