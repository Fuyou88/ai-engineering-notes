# 从 Figma 还原复盘到 Agent Skill 工程化设计（附 Skill）

## 摘要

一次看似普通的 Figma UI 实现任务，暴露出 Agent 在复杂前端还原中的典型问题：只看局部节点、漏状态、改错目标、CSS 未命中第三方组件内部 DOM、用 type-check 代替视觉验收。围绕这些问题，我们逐步将 `figma-design` Skill 从“经验提示”演化为“可执行工作流”：先像前端开发工程师一样完成需求分析、组件设计、参数编码和视觉验收，再进一步沉淀出主入口路由、分层 Reference、产物门禁、DOM Diff 验收、Context Hygiene 和 Token 经济性。

这次演化进一步沉淀出通用的 Agent Skill 设计原则：**Skill 的价值不是让模型知道更多，而是让模型按正确顺序产出正确证据。**

## 定位与边界

`figma-design` Skill 不替代 Figma Dev Mode、Code Connect、Playwright、Chromatic 等现成工具。前者提供设计稿读取、组件映射或视觉回归等基础能力，`figma-design` 要补齐的是 Agent 在复杂 UI 落地过程中的执行顺序、证据产物和验收门禁：什么时候先做整图/状态扫描，什么时候提取 Blueprint，什么时候做组件边界设计，什么时候必须用浏览器 DOM Diff 验证。

## 背景：一次 Figma 还原任务暴露的系统问题

本次演化源于一次复杂配置页的 Figma 还原任务。页面同时包含表单配置、用户选择、状态化列表、弹窗、空态/数据态、手动操作和多端展示等内容。任务看起来是 UI 实现，但实际横跨设计稿理解、组件复用边界、状态建模、第三方组件样式覆盖和视觉验收。

需求本身并不罕见，但执行过程中出现了大量人工校准：

- 某些配置项不是简单新增，而是需要和原有配置区重新组织。
- 关联配置项不应独立成块，而应尊重设计稿中的父子结构和视觉分组。
- 选择态、处理态、空态、数据态、完成态等状态需要完整覆盖。
- 通用选择组件可以复用，但不能耦合当前业务的处理状态。
- Figma 中 modal、tabs、list、footer、button、avatar 等盒模型需要 1:1 对齐。
- Type-check 通过并不能证明视觉正确。

最终发现，问题不只是某个样式写错，而是 Agent 缺少稳定执行复杂 Figma 实现的工程流程。

## 最初的设计理念：让 Agent 像前端工程师一样着手

`figma-design` Skill 最初的工程化思路，并不是先讨论 Token 经济性或 Prompt 结构，而是回到一个前端开发工程师面对设计稿时的自然工作方式。

一个靠谱的前端工程师不会拿到某个 Figma 节点就直接写 CSS。面对复杂设计稿时，通常会先建立一个工作框架，至少包括这些关键动作：

1. **整体理解设计稿和需求。** 粗看所有 Figma layer 和 UI 稿，确认页面范围、模块归属、状态、交互、弹窗、空态、数据态、多端差异，并把这些内容整理成验收清单。
2. **做组件和数据边界设计。** 分析哪些已有组件可以复用，哪些能力来自组件库，哪些需要业务 wrapper，哪些逻辑不能污染公共组件；同时确认数据来源、状态流转和保存/校验边界。
3. **细化实现参数。** 在结构和边界确定后，再进入布局、盒模型、间距、字体、颜色、icon、资产、第三方组件样式覆盖等细节编码。
4. **进行验证和回归。** 结合视觉对比、DOM Diff、交互验证、状态覆盖和必要的测试，确认实现符合设计和业务契约。

这不是完整前端工程流程的全部，而是 `figma-design` Skill 为避免“看到局部反馈就直接补丁式修改”而提炼出的起始工作框架。

所以 `figma-design` Skill 的第一性目标并不是“更会读 Figma”，而是：

> 让 Agent 按一个前端工程师的工作顺序实现设计稿，而不是看到局部反馈就立即补丁式修改。

后续关于 DOM Diff、Screenshot Preflight、Token 经济性、Context Hygiene 的优化，都是在这条工程流程跑起来之后，针对“验证手段不够具体”“注意力容易分散”“上下文被 raw payload 占满”等问题继续演进出来的。

## 问题复盘

| 问题 | 本次表现 | Skill 缺口 | 优化方向 |
|---|---|---|---|
| 只看局部节点 | 用户指出局部配置区，Agent 只修局部 | 缺整图 Layer / State Coverage | 实现前先做 Coverage Matrix |
| UI 结构拆错 | 关联配置项被做成独立块 | 缺 Parent-Child Ownership Check | 实现前判断节点所属父 frame |
| 漏状态 | 选择态、未选择态、已处理/未处理、空态等多轮补漏 | State Matrix 不强制 | Full/Standard 任务必须列状态矩阵 |
| 污染共享组件风险 | 通用选择组件可能耦合业务处理状态 | 缺共享组件边界检查 | 业务状态放 wrapper，公共组件保持纯净 |
| 未做有效 DOM Diff | 多次用 type-check/test 代替视觉验收 | Gate 有，但执行路径不够具体 | 强化视觉任务完成定义 |
| 截图明显不符但未拦截 | 输入框/选择框大小肉眼明显不对 | 缺 Screenshot Preflight | 截图作为低成本预筛 |
| 只改 wrapper | UI Library 内部 DOM 没命中 | 第三方控件内部 DOM 要求不够明确 | 补 Third-party Internal DOM Diff |
| 改错目标 | 用户框的是 controlGroup，却改了 label | 没要求用户指出目标必须纳入 Diff | User-Pointed Target Rule |
| 默认值不一致 | UI 显示默认时间值，校验按另一套默认值计算 | Data/UI consistency 偏抽象 | Display / Save / Validate 同源 |
| 错误文案逻辑混乱 | 输入过程错误和计算错误混在一起 | State Matrix 未要求业务触发条件 | 增加 error trigger / source |

这些问题共同说明：只告诉 Agent “严格按 Figma 实现”并不够，必须把“严格”转成可执行路径和可检查产物。

## figma-design Skill 的工程演化

整个演化过程可以理解为两层：第一层是把 Figma 实现还原为前端工程师的工作流；第二层才是把这个工作流进一步 Skill 化，解决验证、上下文和门禁问题。

### 阶段一：从经验提示到 Extract / Verify 方法论

早期 `figma-design` Skill 已经具备一些正确能力：

- 使用 `get_metadata` 获取精确尺寸。
- 使用 `get_design_context` 理解语义和样式线索。
- 使用 `get_screenshot` 获取视觉基线。
- 提供 Icon Inventory、Third-party Component Audit、Browser DOM Measurement、Diff Report 等能力。
- 强调不要从 Tailwind 类名推断尺寸。

但它更像一份经验说明书，缺少强制产物和阶段门禁。模型可能“看过规则”，但执行时仍跳步骤。

### 阶段二：主入口变成路由器

随后将 `SKILL.md` 改造成薄主入口，只保留：

- Mission
- Complexity Routing
- Progressive Reference Loading
- Non-negotiable Gates
- Reference Router
- Final Delivery Contract

细节下沉到 references：

```text
figma-design/
├── SKILL.md
└── references/
    ├── intake.md
    ├── extract-blueprint.md
    ├── architecture.md
    ├── implementation.md
    ├── verify.md
    ├── modal-list-audit.md
    ├── icon-asset-rules.md
    ├── templates.md
    └── project-rules.md
```

这一步的核心变化是：

> `SKILL.md` 是路由器，不是知识库。

主入口负责判断当前任务是 Simple、Standard 还是 Full，并决定要读取哪些 references。

### 阶段三：产物门禁替代抽象口号

为了避免“看过规则但没执行”，Skill 中加入了 Gate Checklist。

编码前门禁：

```markdown
Before coding:
- [ ] Layer Coverage Matrix exists
- [ ] State Coverage Matrix exists
- [ ] Dimension Spec exists
- [ ] Component Mapping exists
- [ ] Third-party DOM Audit exists
- [ ] Icon & Asset Inventory exists
- [ ] Shared Component Pollution Check completed
- [ ] Data Source Consistency Check completed
```

完成前门禁：

```markdown
Before completion:
- [ ] DOM Diff Report exists
- [ ] Critical diffs fixed
- [ ] Major diffs fixed or explicitly accepted
- [ ] State Coverage verified
- [ ] Remaining Deviations listed
- [ ] Tests/type-check recorded
```

这一步沉淀出的关键经验是：

> 抽象口号不可靠，必须转成可检查产物。

例如：

| 抽象口号 | 可执行产物 |
|---|---|
| 不要漏状态 | State Coverage Matrix |
| 不要猜尺寸 | Dimension Spec |
| 不污染组件 | Shared Component Pollution Check |
| 保证质量 | DOM Diff Report |
| 注意 Token 经济性 | Progressive Loading + Payload Hygiene |

### 阶段四：视觉验收链路具体化

后续又发现：只有“完成前必须 DOM Diff”还不够，因为执行路径仍可能含糊。于是补充了更具体的视觉验收链路。

#### Screenshot Preflight

截图用于低成本发现明显视觉错误，但不能替代 DOM Diff。

```markdown
Screenshot comparison is a preflight, not completion.

Rules:
- If screenshot visibly differs from Figma, do not claim completion.
- Use the screenshot to identify the wrong target, then measure that target with DOM Diff.
- Screenshot can identify the problem; DOM Diff must identify the cause.
- Type-check, tests, and builds never count as visual verification.
```

#### Third-party Internal DOM Diff

当目标由 UI Library 实现时，必须同时测业务 wrapper 和真实内部 DOM。

```markdown
Required targets:
- Input/InputNumber: wrapper, actual input, prefix/suffix, stepper controls if present.
- Select: root, selector, selection item, arrow, dropdown trigger.
- Checkbox/Radio: wrapper, inner box, label text, adjacent help/info icon.
- DatePicker: input wrapper, actual input, suffix icon, popup trigger.
```

原因是第三方组件的真实视觉元素往往不在业务 wrapper 上。只测 wrapper，不能证明 input、selector、icon、border 等细节正确。

#### User-Pointed Target Rule

如果用户提供了截图区域、DOM selector、可见文本或具体描述，该目标必须进入 DOM Diff。

```markdown
Do not verify a nearby parent, sibling, or different state as a substitute.
```

这条规则防止 Agent 修了“附近元素”，却没有修用户真正指出的问题。

#### UI Library Override Rules

编码阶段也要避免无效 CSS：

```markdown
Before overriding UI library styles:
- inspect the rendered DOM hierarchy or known component DOM contract;
- decide whether to style the business wrapper, the library root, or internal elements;
- verify the selector actually matches the rendered DOM.
```

对于 CSS Modules：

- scoped business class 不保证影响 UI Library 内部节点。
- 需要时使用最小范围的 `:global(...)`。
- Figma 要求固定组件尺寸时，优先使用 domain wrapper 承载约束。

#### Display / Save / Validate Consistency

表单类 UI 中，显示值、保存值、校验值、默认值必须来自同一状态模型。

需要检查：

- displayed default value
- validation value
- submit value
- reload restore value
- error trigger
- error source

这条规则防止“UI 看起来是一个默认值，但校验和提交走了另一个默认值”。

### 阶段五：Context Hygiene 与 Token 经济性

随着 Skill 越来越完整，又出现新风险：Skill 本身变重，导致简单任务也读取大量无关规则。

因此增加了 Context Hygiene：

- Simple 任务只读最小 extract + verify。
- Standard 任务按 intake → extract → architecture → implementation → verify 渐进读取。
- Full 任务先做全局 inventory，再对 in-scope 节点深挖。

同时加入 Figma Payload Hygiene：

- 不把大型 metadata 原文贴进上下文。
- 不贴 screenshot base64。
- raw payload 尽量保存为本地文件或工具结果。
- 先生成 compact summary。
- 复访节点先看 summary，必要时再打开 raw metadata。

核心原则是：

> Token 经济性不是少看设计，而是少搬运原始 payload。

## 从 figma-design 提炼出的 Agent Skill 设计原则

这次 `figma-design` 的演化，不只是在优化一个 Figma 实现 Skill，也把 [Agent 工程实践相关 KM](#) 中关于 Skill 设计的原则具体化到了一个真实案例中。下面这些原则，都是在这次 Figma 还原任务的问题和修复中被反复验证的。

### 1. Skill 不是知识堆叠，而是工作流系统

一个好的 Skill 应明确：

- 何时触发
- 如何路由
- 每阶段读取什么
- 每阶段产出什么
- 何时不能继续
- 如何验证完成

它的价值不是让模型“知道更多”，而是让模型“按正确顺序产出正确证据”。

### 2. 主入口是路由器，不是知识库

`SKILL.md` 应尽量薄，只保留触发、路由、流程、门禁和 reference 索引。细节、模板、反模式、项目知识应下沉到 references。

### 3. 保留能力，但不要默认加载

重构 Skill 时不应为了省 Token 删除有效能力。正确做法是：能力完整保留，但按任务阶段和复杂度渐进读取。

> 保留能力 ≠ 默认加载能力。

### 4. 复杂度路由避免一刀切 SOP

| 复杂度 | 场景 | 路径 |
|---|---|---|
| Simple | 单点修复、单 icon、单 button | 最小提取 + 最小验证 |
| Standard | 单组件多状态、单弹窗、单表单段 | intake + extract + architecture + verify |
| Full | 页面级、多状态、多端、多弹窗 | 完整 SOP |

Simple 任务不要默认背完整 SOP，复杂任务也不能跳过 coverage / extract / verify。

### 5. Reference 单一职责，但目录不固定

一开始我们差点把 `intake / extract / architecture / implementation / verify` 当成通用结构。后来意识到，这仍然过拟合阶段型 Skill。

最终原则是：Reference 可以按阶段、领域、工具、产物或项目拆分。文件名不是固定标准，关键是职责边界清晰。

| 拆分方式 | 示例 |
|---|---|
| 按阶段 | `intake.md` / `extract.md` / `verify.md` |
| 按领域 | `api.md` / `schema.md` / `policy.md` |
| 按工具 | `cli.md` / `browser.md` / `database.md` |
| 按产物 | `templates.md` / `examples.md` |
| 按项目 | `project-rules.md` |

### 6. 共享能力污染检查

Figma 任务中的通用选择组件复用问题，抽象出一个通用原则：业务状态不要泄漏进共享能力。

共享能力可以是组件、工具、服务、脚本、schema、prompt 模板等。常见策略包括：

- Reuse as-is
- Wrap
- Extend generic capability
- Fork domain-specific version
- Do not implement

### 7. 数据同源检查

状态化列表中曾出现“统计数量和列表内容不一致”的风险。这说明 count、list、empty、action availability 必须来自同一数据源或同一契约。

表单类 UI 也一样：display、save、validate、default、error source 必须同源。

### 8. 验证是独立阶段

不同任务需要不同验证产物：

| 任务类型 | 验证方式 |
|---|---|
| UI | DOM diff / screenshot comparison / interaction check |
| API | contract test / request-response check |
| Data | consistency check / migration dry-run |
| Refactor | impact analysis / tests |
| Docs | completeness checklist |
| Logs / search | query replay / sample validation |

对于 Figma UI，实现完成的定义是：

> No DOM Diff Report = not done.

### 9. 失败和降级必须显式

可以降级，但必须标注降级，不能假装完整执行。

```markdown
If tool X unavailable:
- record reason
- use fallback Y
- mark limitations
- do not claim full completion
```

## 总结

这次 `figma-design` Skill 的演化，本质上不是一次简单的提示词优化，而是一次从经验到工程化方法的沉淀。

最初的问题是：Agent 可以写代码，但容易跳过理解、范围、状态、验证这些关键步骤。最终的解决方案是：用路由、分层 references、结构化产物和强制门禁，把“应该注意”变成“必须产出”。

其中最关键的经验是：Skill 的价值不是让模型知道更多，而是让模型按正确顺序产出正确证据；Type-check 证明代码能编译，不证明视觉被还原；Token 经济性也不是少看设计，而是少搬运原始 payload。

这套思路不仅适用于 Figma 设计稿实现，也适用于 API 开发、日志分析、数据修复、文档生成、重构评审等需要 Agent 稳定执行复杂任务的场景。

## 阅读推荐

本文是技能设计方法论（下篇·案例篇），通过一个真实 Skill 的 5 阶段演化展示设计原则如何落地。上篇 [《Agent 工程实践相关》](#) 从通用方法论角度，系统阐述 12 条可执行的设计原则：复杂度路由、产物门禁、Token 经济性、共享能力污染检查、数据同源等，适合作为 Skill 设计/评审的工具手册。两篇互为补充：上篇建方法论框架，下篇看实践案例。

## Skill 实现

[figma-design](#) — 本文所述原则的落地 Skill，内置复杂度路由、Blueprint 提取、DOM Diff 验收门禁、Screenshot Preflight 等机制。
