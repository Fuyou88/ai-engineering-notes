---
name: figma-design
description: Figma 设计稿的应用级前端落地 SOP。用于从 Figma URL/nodeId 实现、提取、核对或验收 UI：先做整图/状态扫描，产出 Blueprint（Layer/State Matrix、Dimension Spec、Icon/Asset Inventory、Component Mapping），再编码，最后用浏览器 DOM Diff 验证还原度。触发词："提取 Figma"、"Figma 实现准备"、"figma extract"、"figma design"、"对比 Figma"、"设计还原验证"、"UI 验证"、"figma verify"、"Implement this design from Figma"。
---

# Figma Design Engineering SOP

这是应用级 Figma → 前端工程落地流程，不是单节点样式摘抄。你必须像前端负责人一样工作：理解完整视觉稿，枚举状态，提取精确 Blueprint，做组件解耦设计，编码，然后用浏览器 DOM 测量验证。

## Core Principle

> **NEVER infer dimensions. ALWAYS measure.**

- 绝对尺寸、gap、padding、盒模型：以 `get_metadata` 的 `x/y/width/height` 为准。
- Tailwind / Code Connect / design context：只用于理解语义、颜色、字重、组件意图，不得替代尺寸测量。
- 截图：用于视觉基线和最终对比，不得替代 metadata 或 DOM measurement。
- 完成定义：没有 Browser DOM Diff Report，不得说“完成”。

## Required Tools

- Figma Desktop MCP：`get_design_context`、`get_metadata`、`get_screenshot`，用于 Extract。
- Browser / Chrome DevTools：DOM 检查、`getBoundingClientRect()`、截图，用于 Verify。
- 项目搜索工具：优先 `rg`，用于复用组件、图标、第三方默认样式审计。

## Do Not Use

- 不用于纯产品讨论、设计建议或无需落地代码的视觉点评。
- 不用于普通 React/CSS bug 修复，除非用户要求对齐 Figma 或进行视觉还原验证。
- 不用于只需要翻译、文案润色、代码解释的任务。
- 用户明确要求跳过 Figma/DOM 验证时，不启动完整 SOP；需说明跳过后的验证风险。

## Complexity Routing

| 复杂度 | 适用场景 | 必须读取 | 最低产物 |
|---|---|---|---|
| Simple | 单个 icon/button/text 修复 | `references/extract-blueprint.md`、`references/verify.md` | node-level Dimension Spec + DOM Diff |
| Standard | 单组件多状态、弹窗、列表、表单段落 | `references/intake.md`、`extract-blueprint.md`、`architecture.md`、`implementation.md`、`verify.md` | State Matrix + Blueprint + DOM Diff |
| Full | 页面/section/root、多弹窗、多端、多交互 | 全部相关 references | Whole Canvas Coverage + Architecture + Full DOM Diff |

Routing rules:

- 用户提供 section/page/root node，或说“整个视觉稿 / 全部图层 / full design / complete design / 所有状态”时，走 Full。
- 目标包含 icon/statusTips/question/info/image/empty illustration 时，额外读取 `references/icon-asset-rules.md`。
- 目标位于 modal/list/tabs/footer/avatar/button 盒模型内时，额外读取 `references/modal-list-audit.md`。
- 用户反馈视觉偏差时，先判断是 node-level deviation 还是更大 state frame 的一部分；不确定时走 Standard。
- 不确定 Simple / Standard / Full 时，选择更高一级复杂度。

## Progressive Reference Loading

不要默认读取所有 references。按任务类型和阶段渐进读取。

### Simple tasks

用于单节点视觉偏差、单个 icon、button、text、spacing 或 tooltip 修复。

Required:

- `references/extract-blueprint.md`
- `references/verify.md`

Conditional:

- icon / image / empty illustration / asset：读取 `references/icon-asset-rules.md`。
- modal / list / tabs / footer / avatar / button box model：读取 `references/modal-list-audit.md`。
- tooltip / popover：读取 `references/implementation.md` 的 Tooltip / Popover Rules。
- formal artifact output：读取 `references/templates.md`。

### Standard tasks

用于单组件多状态、modal、list、form section 或可复用组件集成。

Read progressively:

1. `references/intake.md`
2. `references/extract-blueprint.md`
3. `references/architecture.md`
4. `references/implementation.md`
5. `references/verify.md`

Conditional:

- modal/list/tabs/footer/avatar/button：读取 `references/modal-list-audit.md`。
- icon/asset/empty illustration：读取 `references/icon-asset-rules.md`。
- formal artifact output：读取 `references/templates.md`。

### Full tasks

用于 root/page/section Figma nodes、多弹窗流、多状态屏幕或多端设计。

Read progressively, not all at once:

1. 先只读 `references/intake.md`，产出 Layer / State Matrix。
2. 再读 `references/extract-blueprint.md`，对 in-scope nodes 提取 Blueprint。
3. 再读 `references/architecture.md`，完成组件映射和数据流。
4. 编码前读 `references/implementation.md`。
5. 完成前读 `references/verify.md`。

Conditional:

- 包含 modal/list/tabs/footer/avatar/button：读取 `references/modal-list-audit.md`。
- 包含 icon/asset/empty illustration：读取 `references/icon-asset-rules.md`。
- producing formal artifact：读取 `references/templates.md`。

## Routing

### Implement from Figma

必须按顺序执行：

1. 读取 `references/intake.md`，产出 Layer Coverage Matrix、State Coverage Matrix、Scope Decision。
2. 读取 `references/extract-blueprint.md`，调用 Figma MCP，产出 Dimension Spec、Box Model Check、Icon Inventory、Asset Inventory、Typography Spec。
3. 读取 `references/architecture.md`，产出 Component Mapping、Reuse Decision、Data Flow、Third-party DOM Audit Plan。
4. 读取 `references/implementation.md`，再开始编码。
5. 如果涉及 modal/list/tabs/footer/avatar，读取 `references/modal-list-audit.md` 并做专项测量。
6. 读取 `references/verify.md`，用浏览器 DOM measurement 产出 DOM Diff Report。
7. 按 diff 修复并复测，输出剩余偏差和测试/type-check 结果。

### Verify against Figma

1. 加载已有 Blueprint；若缺失，先运行最小 Extract。
2. 读取 `references/verify.md`。
3. 测量 DOM，生成 Diff Report。
4. 先给出量化差异，再修复；禁止靠肉眼猜测修改。

### Fix user-reported visual deviation

1. 定位用户指出的 Figma node 和浏览器 DOM 元素。
2. 提取该 node metadata / screenshot；如属于父 frame，先做 Parent-Child Ownership Check。
3. 用 DevTools 测量当前 DOM。
4. 输出 Figma vs DOM 差异。
5. 修复后重新测量，直到 Critical 清零。

## Non-negotiable Gates

### Gate 1 — No Code Before Blueprint

写代码或改代码前，必须已有与复杂度匹配的 Blueprint。

Simple 修复最小产物：

1. Node-level Dimension Spec
2. Verify target（DOM selector / accessible target / 状态触发方式）
3. Component Mapping mini（reuse / wrapper / direct edit）
4. Icon / Asset mini inventory（仅当涉及 icon / image / asset）

Standard / Full 必须已有：

1. Layer Coverage Matrix
2. State Coverage Matrix
3. Dimension Spec
4. Component Mapping
5. Third-party DOM Audit Plan
6. Icon & Asset Inventory

任一必需项缺失：停止编码，先提取。

### Gate 2 — No Completion Before DOM Diff

声称完成前，必须已有：

1. Browser DOM Diff Report
2. State Coverage Verification
3. Remaining Deviations List
4. Test / type-check 结果
5. Adjacent Box Boundary Diff：当任务涉及 spacing / gap / padding / alignment / visual adjacency 时，DOM Diff Report 必须包含相关相邻盒模型边界测量；仅检查单元素尺寸或 CSS gap 值视为验证未完成。

无 DOM Diff Report = 未完成。Critical diffs 必须修复；Major diffs 必须修复或明确原因；Minor diffs 可说明接受。

### Gate 3 — Shared Component Pollution Check

不得为业务特定状态污染共享组件。每个复用组件必须选择一种策略：

- Reuse as-is
- Wrap in domain component
- Extend through generic props
- Fork into domain component
- Do not implement

业务数据、业务状态、业务 API 不得泄漏进通用选人、通用 Modal、通用 List 等共享组件，除非用户明确批准。

### Gate 4 — Data/UI Consistency

count、list、empty、action availability 必须来自同一数据源或同一契约。禁止用 selected count、mock count、fallback array 伪造业务状态。

## API Budget & Failure Handling

- 默认 Figma API budget：10 次；每次调用前检查剩余额度；多节点串行，禁止并发。
- 同一会话同一 nodeId 使用缓存，标注 `(cached)`。
- `get_design_context` 被 Code Connect prompt 阻断、MCP 不可用、截图超时、rate limit 时，记录 degraded mode，不得假装已完整读取。
- 若 rate limit exceeded：立即停止后续 Figma 调用，输出已收集信息，并建议用户提供截图或等待配额恢复。
- 降级可以继续分析，但所有近似值必须标注 `⚠️ approximate`，不得作为完成依据。

## Reference Router

- `references/intake.md`：整图扫描、状态覆盖、范围决策、复杂度路由。
- `references/extract-blueprint.md`：Figma MCP 调用顺序、尺寸/盒模型/图标/资源/字体提取。
- `references/architecture.md`：组件复用、解耦、数据流、第三方 DOM 审计。
- `references/implementation.md`：编码阶段规则，多状态、Modal/List、Tabs、Tooltip、Button。
- `references/verify.md`：DOM measurement 脚本、Diff Report、Severity、Fix Loop。
- `references/modal-list-audit.md`：Modal/List/Tabs/Footer/Button/Avatar 专项审计。
- `references/icon-asset-rules.md`：图标映射、图片资源、暗黑主题、资源决策。
- `references/templates.md`：标准产物模板。
- `references/project-rules.md`：项目特定约束、反模式、迁移自旧 skill 的项目知识。

## Subagent Protocol

复杂任务可并行，但主 agent 保留最终验收权。派发前必须提供结构化输入：

```markdown
Task:
Figma nodes:
Dimension Spec:
State Matrix:
Allowed files:
Do not modify:
Expected output:
```

- Intake / Extract / Architecture / Verify agent 默认不写代码。
- Implementation agent 不得自行扩大设计范围。
- Verify agent 不得修代码，除非主 agent 明确授权。
- Main agent 不得跳过 Extract/Architecture 直接派发 Implementation。

## Final Delivery Contract

最终回答必须包含：

- Blueprint 覆盖范围：已覆盖节点/状态，未覆盖节点/状态。
- 关键实现文件。
- DOM Diff Summary：Critical/Major/Minor 数量与处理结果。
- Adjacent Box Boundary Diff Summary：涉及 spacing / gap / padding / alignment / visual adjacency 时必须给出。
- State Coverage Result：每个目标状态的验证结果。
- Remaining Deviations：未消除偏差及是否接受。
- Test/type-check result：测试或类型检查结果；若未运行，说明原因。
