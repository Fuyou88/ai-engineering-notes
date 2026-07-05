# Project Rules & Anti-patterns

迁移自旧 `figma-design` skill 的项目知识和通用反模式。使用时还必须遵守仓库 `AGENTS.md`。

## UI Library

- PC：`@bedrock/components`
- Mobile：`@bedrock/mobile`，CSS 前缀常见为 `adm-`
- 样式：CSS Modules + Less (`.module.less`)
- 覆盖第三方组件：CSS Modules `:global { ... }`
- Token：优先项目已有 `var(--color-*)`、`var(--space-*)`、`var(--radius-*)`、`var(--font-size-*)`

## Reusable Components

| Component | Path | Usage |
|---|---|---|
| `PageNavFrame` | `components/page-nav/page-nav-frame.tsx` | 三栏导航栏 |
| `CircleButton` | `components/common/circle-button/index.tsx` | 圆形 icon 按钮 |
| `PageNav` | `components/page-nav/index.tsx` | 高层导航栏 |

搜索项目时以实际 repo 结果为准，上表只是起点。

## Anti-patterns

| Forbidden | Required |
|---|---|
| 从 Tailwind 推算容器尺寸 | `get_metadata` 读精确 px |
| 跳过 gap 计算 | 相邻子元素坐标差值 |
| 忽略容器 padding | 首/末子元素位置计算 |
| 只用 `get_design_context` | 串行 context → metadata → screenshot |
| 写完不验证 | Verify 模式 DOM 测量 |
| 直接写 CSS 覆盖 | 先读默认 CSS → diff → 覆盖 |
| 信任 `line-height: normal` 跨平台 | metadata 读 row/text height，显式设置 |
| 跳过盒模型加法 | padding + Σ(child) + Σ(gap) = container |
| 肉眼判断“一致” | 数值对比，≤ 2px 才通常可接受 |
| 批量修完才验证 | 每修一项立即重新测量 |
| 用文本字符代替 SVG icon | import 正确 icon 组件 |
| 多节点并发调用 Figma | 逐个串行，检查 budget |
| Rate limit 后继续调用 | 立即停止并输出已收集信息 |
| 重复调用同一 nodeId | session 缓存 |
| 无限递归展开 instance | 最大 2 层，每层消耗 budget |
| `getComputedStyle` 测尺寸 | `getBoundingClientRect()` |
| 把页面级 frame bg/shadow/radius 当组件样式 | 先判断父子归属，只取组件自身样式 |
| 业务状态污染共享组件 | 用业务 wrapper 或 generic props |
| count/list 不同源 | 同一契约/response 驱动 UI |
