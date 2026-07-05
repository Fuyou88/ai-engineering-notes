# 12个 Agent Skill 设计原则：从知识堆叠到工作流系统 （附 Skill）

## 摘要

一个好的 Agent Skill 不是“知识堆叠文档”，而是让模型稳定执行某类任务的**工作流系统**。它应明确：何时触发、如何路由、每阶段读取什么、产出什么、何时不能继续、如何验证完成。

核心原则：**主入口轻，阶段清晰，能力完整，按需加载；少写抽象口号，多写产物门禁；用可执行流程替代价值判断，用验收闭环证明完成。**

## 背景与范围

Agent Skill 通常由 `SKILL.md` 作为主入口，并按需包含 `references/`、`scripts/`、`assets/` 等资源，用来让 Agent 在某类任务上稳定执行。

本文是通用 Skill 设计和评审原则，适用于大多数需要稳定执行流程的 Agent Skill：编码、调试、设计稿实现、文档编排、API 开发、数据修复、日志分析等。

并不是所有提示词都需要沉淀为 Skill。一次性、低复用、低风险任务，直接用普通提示词或短文档即可，避免过度工程化。

适用场景：

- 设计一个新 skill。
- 重构一个已有 skill。
- 评审 skill 是否大而全、是否 token 浪费、是否可执行。
- 将经验型提示词沉淀为可复用 skill。

## 方案总览

### Skill 的系统视图

```text
User task
  → SKILL.md 主入口识别触发和任务模式
  → 按复杂度和阶段读取 references
  → 每阶段产出结构化产物
  → 产物门禁决定是否进入下一步
  → Verify / Delivery 产物证明完成
```

### 目录结构选择规则

Skill 目录不应固定套用同一种结构。目录应由任务类型、复杂度、是否需要确定性脚本、是否有输出资产决定。最小通用结构如下：

```text
skill/
├─ SKILL.md                 # 必需：触发、路由、最小 SOP、门禁
├─ references/              # 可选：按需加载的阶段/领域知识
│  ├─ workflow.md           # 可选：主流程或阶段 SOP
│  ├─ <phase-or-domain>.md  # 可选：按阶段或领域拆分
│  ├─ templates.md          # 可选：产物模板
│  └─ project-rules.md      # 可选：项目特定知识和反模式
├─ scripts/                 # 可选：确定性工具、转换、校验脚本
└─ assets/                  # 可选：模板、字体、图片、示例资源
```

常见拆分模式：

- **阶段型 skill**：适合工程实现、设计稿落地、数据修复等线性流程，可拆为 `intake.md` / `extract.md` / `architecture.md` / `implementation.md` / `verify.md`。
- **工具型 skill**：以 `scripts/` 为主，搭配少量 `references/usage.md` 说明输入、输出和限制。
- **知识型 skill**：按领域或对象拆分，例如 `references/api.md` / `schema.md` / `policy.md`。
- **格式处理 skill**：通常组合 `scripts/`、`assets/` 和 `references/format-rules.md`。
- **轻量 skill**：只有 `SKILL.md` 也可以，不因“规范化”强拆 references。

### 阶段型 Skill 的推荐主流程

```text
Intake → Extract → Architecture → Implementation → Verify → Delivery
```

| 阶段 | 目标 | 典型产物 |
|---|---|---|
| Intake | 理解需求、范围、状态 | Scope / Coverage Matrix |
| Extract | 获取事实 | Spec / Contract / Dimension / Data Map |
| Architecture | 设计边界与方案 | Component Mapping / Data Flow / Impact |
| Implementation | 执行变更 | Code / Config / Docs |
| Verify | 量化验收 | Diff Report / Test Result |
| Delivery | 交付总结 | Summary / Risks / Deviations |

## 核心设计原则

### 1. 主入口是路由器，不是知识库

`SKILL.md` 应尽量薄，只保留：

- 触发条件
- 任务模式识别
- 复杂度路由
- 阶段流程
- 硬门禁
- reference 索引

细节、模板、反模式、项目知识应下沉到 references。

反模式：一个 `SKILL.md` 里放所有背景、所有规则、所有模板、所有项目知识。

> 示例（Figma 设计稿实现类 skill）：
> 设计稿实现通常同时涉及提取、组件设计、编码和验收。如果这些内容都塞进主入口，模型在修一个按钮时也会读到大量无关细节。更好的方式是主入口只判断当前是 Extract、Implement 还是 Verify，再路由到对应 reference。

### 2. 保留能力，但不要默认全量加载

重构 skill 时，不应为了 token 经济性删除能力。正确做法是：能力完整保留，但按任务阶段和复杂度渐进读取。

| 能力类型 | 推荐位置 |
|---|---|
| 核心工具说明 | 对应阶段 reference |
| 项目知识 | `project-rules.md` |
| 输出模板 | `templates.md` |
| 反模式 | appendix / project rules |
| 复杂专项 | 单独 reference，按需读取 |

关键原则：**保留能力 ≠ 默认加载能力。**

> 示例（图标和资源处理类能力）：
> Icon 映射、空态插画、复杂资源导出都是有效能力，但不是每个任务都需要。它们应保留在独立 reference 中，只有任务涉及 icon、asset 或 empty illustration 时再读取。

### 3. 用产物门禁替代抽象口号

抽象口号常见但低效，例如：

- 保证质量
- 注意完整性
- 遵循最佳实践
- 保持可维护
- 不要降低覆盖

这些话不一定错，但通常不可操作、不可验收、容易占用上下文并稀释注意力。

更好的方式是将其转成可执行产物。

| 抽象口号 | 可执行规则 / 产物 |
|---|---|
| 不要漏状态 | State Coverage Matrix |
| 不要猜 | Fact Spec / Dimension Spec / Contract |
| 保证质量 | Verify Report |
| 不污染组件 | Shared Component Pollution Check |
| 注意 token 经济性 | Progressive Loading + Raw Payload Hygiene |
| 不要说完成太早 | Completion Gate |
| 保持可维护 | Component Mapping / Data Flow |

> 示例（设计覆盖类规则）：
> 与其写“不要降低设计覆盖”，不如要求 Full 任务先输出 Layer Coverage Matrix 和 State Coverage Matrix。这样读者和模型都能直接检查：哪些页面、弹窗、空态、数据态已经被覆盖。

判断一个句子是否是噪音：

| 问题 | 若答案为“否”，该句就是噪音候选 |
|---|---|
| 是否告诉模型下一步做什么？ | 否 |
| 是否产生可检查产物？ | 否 |
| 是否影响路由或决策？ | 否 |
| 是否可被更具体规则替代？ | 是 |

### 4. 按复杂度路由

不要让所有任务都走完整 SOP，也不要让复杂任务走轻量路径。

| 复杂度 | 场景 | 路径 |
|---|---|---|
| Simple | 单点修复、单字段、单按钮、单事实查询 | 最小提取 + 最小验证 |
| Standard | 单组件多状态、单接口、单模块、单弹窗 | intake + extract + implementation + verify |
| Full | 页面级、多模块、多端、多状态、跨系统 | 完整 SOP |

规则：不确定复杂度时升一级，但 simple 任务不要默认读完整 SOP。

> 示例（局部视觉偏差修复）：
> 如果只是一个 tooltip 图标不一致，可以走 Simple：提取该节点尺寸和图标名，做 DOM diff 后修复。若该图标位于复杂弹窗 header 中，并影响弹窗布局，就应升级到 Standard，补充读取弹窗/列表专项规则。

### 5. Token 经济性靠渐进式读取，而不是减少理解

省 token 的对象是 raw payload 和无关规则，不是任务理解本身。

| 方法 | 说明 |
|---|---|
| Progressive Reference Loading | 按阶段读取 reference |
| Two-pass Extract | 先全局 inventory，再局部深挖 |
| Summary-first Cache | 大 payload 转成摘要 |
| Template on demand | 正式输出时才读模板 |
| Compact Verify | 只测关键字段 |
| Raw Payload Hygiene | 不粘贴 base64、大 JSON、大 XML、大日志原文 |

错误做法：为了省 token 不看完整范围、不读关键契约、不做验证。

> 示例（大 payload 处理）：
> 对大型设计稿、日志结果或接口 schema，不应为了省 token 跳过全局扫描；应先生成节点索引、状态矩阵或接口摘要，再只对 in-scope 项深挖。这样既不漏范围，也不把整份 raw payload 塞进上下文。

### 6. Reference 单一职责

每个 Reference 应有单一职责，可按阶段、领域、工具或产物类型拆分，避免互相重复。文件名不是固定标准，关键是职责边界清晰。

| 拆分方式 | 示例文件 | 负责 |
|---|---|
| 按阶段 | `intake.md` / `extract.md` / `verify.md` | 需求理解、事实提取、验收验证 |
| 按领域 | `api.md` / `schema.md` / `policy.md` | API、数据结构、规则约束 |
| 按工具 | `cli.md` / `browser.md` / `database.md` | 工具调用流程和限制 |
| 按产物 | `templates.md` / `examples.md` | 输出模板和示例 |
| 按项目 | `project-rules.md` | 项目特定约束和反模式 |

反模式：一个 Reference 同时承担流程、模板、项目规则和反模式；或多个 Reference 重复同一套规则，导致模型不知道以哪个为准。

### 7. 子 agent 协作要结构化

如果 skill 支持子 agent，必须定义交接协议，而不是只给自然语言任务。

推荐模板：

```markdown
Task:
Input artifacts:
Relevant files:
Allowed write scope:
Do not modify:
Expected output:
Verification:
```

原则：

- 子 agent 不决定范围。
- 子 agent 不跳阶段。
- 主 agent 保留最终验收。
- 输出必须产物化，不只说“已完成”。

### 8. 共享能力污染检查

任何涉及复用能力的 skill 都应有边界检查。共享能力包括组件、工具、服务、脚本、schema、prompt 模板等。

| 策略 | 说明 |
|---|---|
| Reuse as-is | 直接复用 |
| Wrap | 外层业务封装 |
| Extend generic capability | 只加通用能力 |
| Fork domain-specific version | 业务域复制/新建 |
| Do not implement | 明确不做 |

原则：业务特定状态不要泄漏进共享能力。

> 示例（业务组件复用）：
> 通用选人组件只应负责选择用户、群组或部门，不应知道“已投/未投”这类业务状态。业务状态可以放在外层业务 wrapper 中展示，公共组件保持可复用。

### 9. 数据同源检查

UI、API、报表、修复脚本中常见错误是 count、list、empty、action 来源不一致。

建议建立检查表：

| UI / Output Item | Data Source | Same Response / Contract? | Empty / Error Logic | Action Availability |
|---|---|---|---|---|

原则：

- count 和 list 应来自同一契约或同一 response。
- empty state 不能由 fallback 假数据驱动。
- action availability 不能和数据状态冲突。

> 示例（列表和统计 UI）：
> 如果 tab 上显示“未处理 5 人”，但列表为空，读者会立即感知不一致。skill 应要求 count、list、empty state 都来自同一个接口响应或同一个数据契约。

### 10. 验证是独立阶段

实现完成不等于任务完成，必须有验证产物。

| 任务类型 | 验证方式 |
|---|---|
| UI | DOM diff / screenshot comparison / interaction check |
| API | contract test / request-response check |
| Data | consistency check / migration dry-run |
| Refactor | impact analysis / tests |
| Docs | completeness checklist |
| Logs / search | query replay / sample validation |

> 示例（UI 实现验收）：
> Type-check 通过只能说明代码类型正确，不代表视觉还原完成。对于 UI skill，完成前应有 DOM diff、截图对比或交互验证等验收产物。

完成门禁示例：

```markdown
Before completion:
- [ ] Verification artifact exists
- [ ] Critical issues fixed
- [ ] Remaining deviations listed
```

### 11. 失败和降级必须显式

Skill 应明确工具失败、权限不足、数据缺失时怎么做。

```markdown
## Degraded Mode

If tool X unavailable:
- record reason
- use fallback Y
- mark limitations
- do not claim full completion
```

原则：可以降级，但必须标注降级；不能假装完整执行。

> 示例（工具不可用）：
> 如果设计工具、日志平台或搜索接口不可用，可以改用截图、缓存或样本数据继续分析，但最终结论必须标注 degraded mode，不能声称已经完整验证。

### 12. KISS：只为高频/高风险场景加默认规则

不要因为一次边缘失败就加复杂默认流程。

| 问题 | 若答案为“否”，不要加默认规则 |
|---|---|
| 是否高频？ | 否 |
| 是否高风险？ | 否 |
| 是否可通过现有规则覆盖？ | 是 |
| 是否会增加 simple 任务成本？ | 是 |
| 是否能产生明确产物？ | 否 |

## 横切关注点

### Token 经济性

- 主入口薄，减少默认上下文。
- references 按阶段读取。
- 大 payload 先摘要，再按需打开原文。
- 模板仅在正式输出时读取。

### 可维护性

- 不删除旧能力，只迁移分层。
- 产物模板集中管理。
- 项目特定知识和通用流程分离。

### 可验证性

- 每个关键阶段都有产物。
- 完成必须有验证证据。
- 失败/降级必须显式记录。

## 实施与验证

### Skill 重构公式

```text
1. 盘点现有有效能力，不删除
2. 提炼主入口：触发 + 路由 + 门禁
3. 按阶段拆 references
4. 把抽象口号转为产物清单
5. 加 Complexity Routing
6. 加 Context Hygiene
7. 加 Verify / Delivery Gate
8. 明确不做什么
9. 保留项目特定知识为 appendix
10. 用 KISS 删除低收益复杂机制
```

### Skill 质量评审清单

| 问题 | 通过标准 |
|---|---|
| 触发是否清晰？ | 知道何时用/不用 |
| 主入口是否薄？ | 不承载全部细节 |
| 是否有复杂度路由？ | Simple/Standard/Full 或等价 |
| 是否阶段化？ | 每阶段有输入/输出 |
| 是否有产物门禁？ | 没产物不能进入下一步 |
| 是否避免抽象口号？ | 规则可执行、可验收 |
| 是否保留能力但按需加载？ | 能力完整，默认轻量 |
| 是否有 token 经济性？ | 渐进读取、摘要优先 |
| 是否有验证闭环？ | 完成前有验证产物 |
| 是否有降级策略？ | 工具失败不假装完成 |
| 是否符合 KISS？ | 不为低频场景加默认复杂度 |

## 总结

一个好 skill 的价值，不是让模型“知道更多”，而是让模型“按正确顺序产出正确证据”。

最终原则：

```text
主入口轻
阶段清晰
能力完整
按需加载
少写口号
多写产物
门禁驱动
验证闭环
KISS 优先
```

## Skill 实现

[skill-design-guide](#) 是本文原则的配套实践工具。它支持 Design / Refactor / Review 三种任务模式，将抽象设计原则转化为可填写、可检查、可交付的产物模板，例如 Skill Worthiness Check、File Structure Decision、Reference Responsibility Check、Scorecard、Abstract-to-Artifact 重写表、能力保有清单、Problems & Noise Table、Verification Strategy 和 Degraded Mode Check。
