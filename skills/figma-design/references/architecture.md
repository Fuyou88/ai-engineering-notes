# Architecture — Component Mapping & Decoupling

用于把 Figma Blueprint 转成工程实现方案，避免重复建设和污染共享组件。

## Inputs

- Intake Matrix
- Extract Blueprint
- 现有代码搜索结果
- 项目约束 / AGENTS.md / 技术方案

## Reusable Component Discovery

1. 用 `rg` 搜索现有同类 UI：表单块、选人、Modal、Tabs、List、Empty、Button、Tooltip、Icon。
2. 阅读组件 props、状态管理、样式边界和调用方。
3. 判断组件是否通用，是否已有业务耦合。

## Component Mapping Template

| Figma Module | Code Component | Strategy | Reason | Do Not |
|---|---|---|---|---|

Strategy：

- Reuse as-is
- Wrap in domain component
- Extend through generic props
- Fork into domain component
- New domain component
- Do not implement

## Shared Component Pollution Check

禁止把业务特定数据或状态塞进共享组件。例如：

- 通用选人组件不得知道“已投票/未投票”。
- 通用 Modal 不得知道某业务接口。
- 通用 List 不得内置业务 empty 文案。

如需扩展共享组件，只能增加通用能力，并必须评估所有调用方；否则使用业务 wrapper。

## Data Flow Template

```markdown
## Data Flow
Source of truth:
- config:
- counts:
- list:
- action status:

Load flow:
- 

Submit flow:
- 

Error mapping:
- 
```

## Data Source Consistency Gate

| UI Item | Data Source | Same Response? | Empty Logic | Action Availability |
|---|---|---|---|---|

规则：count、list、empty、action availability 必须来自同一契约或同一 response；禁止用 fallback count 填业务列表。

## Third-party DOM Audit Plan

第三方组件必须先审计 DOM 和默认 CSS，再写覆盖。

| Component | Default DOM Risk | Required Measurement | Decision |
|---|---|---|---|

常见风险：

- Modal body 默认 padding / overflow / scrollbar wrapper。
- Tabs 默认 ink bar / header padding / line-height。
- Button 默认 min-width / height / border-radius / icon gap。
- Tooltip 默认 arrow / background / max-width。
- Empty 默认插画和 margin。

## Output Gate

进入编码前必须产出：

- Component Mapping
- Shared Component Pollution Check
- Data Flow
- Data Source Consistency Gate
- Third-party DOM Audit Plan
