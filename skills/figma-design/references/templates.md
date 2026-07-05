# Templates

复制以下模板作为 Figma 实现、验证和最终交付的标准产物。

## Figma Intake Report

```markdown
# Figma Intake Report

## Figma Input
- Root node:
- Target app:
- User goal:
- Complexity: simple / standard / full

## Layer Coverage Matrix
| Node ID | Name | Layer Type | Parent Frame | Scenario / State | Target App | Implement? | Code Area | Notes |
|---|---|---|---|---|---|---|---|---|

## State Coverage Matrix
| Component | State | Figma Node | Trigger | Expected UI | Data Source | Implemented By | Verify Target |
|---|---|---|---|---|---|---|---|

## Scope Decision
In scope:
- 

Readonly / partial:
- 

Out of scope:
- 

Open questions:
- 
```

## Implementation Blueprint

```markdown
# Implementation Blueprint

## Dimension Spec
| Node | Element | Parent | x | y | width | height | CSS Target | Notes |
|---|---|---|---:|---:|---:|---:|---|---|

## Box Model Check
| Container | Equation | Expected | Actual Sum | Result |
|---|---|---:|---:|---|

## Typography Spec
| Node | Text | Figma height | Font size | Figma line height | Code decision | Notes |
|---|---|---:|---:|---|---|---|

## Icon Inventory
| Figma Name | Node | Size | Code Icon | Package | Exists? | Usage |
|---|---|---:|---|---|---|---|

## Asset Inventory
| Node | Asset Name | Type | Size | Strategy | Dark Mode? | Notes |
|---|---|---|---:|---|---|---|

## CSS Override List
| Component | Default CSS | Figma | Override | Reason |
|---|---|---|---|---|
```

## Architecture Report

```markdown
# Figma Implementation Architecture

## Component Mapping
| Figma Module | Code Component | Strategy | Reason | Do Not |
|---|---|---|---|---|

## Data Flow
Source of truth:
- config:
- counts:
- list:
- action status:

## Data Source Consistency
| UI Item | Data Source | Same Response? | Empty Logic | Action Availability |
|---|---|---|---|---|

## Third-party DOM Audit Plan
| Component | Default DOM Risk | Required Measurement | Decision |
|---|---|---|---|
```

## Gate Checklist

```markdown
## Gate Checklist

Before coding:
- [ ] Layer Coverage Matrix exists
- [ ] State Coverage Matrix exists
- [ ] Dimension Spec exists
- [ ] Component Mapping exists
- [ ] Third-party DOM Audit exists
- [ ] Icon & Asset Inventory exists
- [ ] Shared Component Pollution Check completed
- [ ] Data Source Consistency Check completed

Before completion:
- [ ] DOM Diff Report exists
- [ ] Adjacent Box Boundary Diff exists when spacing / gap / padding / alignment / visual adjacency is in scope
- [ ] Critical diffs fixed
- [ ] Major diffs fixed or explicitly accepted
- [ ] State Coverage verified
- [ ] Remaining Deviations listed
- [ ] Tests/type-check recorded
```

## Node Summary Template

```markdown
| Field | Value |
|---|---|
| Node ID | |
| Role | page / modal / list / row / button / icon / empty / tooltip |
| Parent | |
| Size | |
| Children | |
| States | |
| Critical Dimensions | |
| Icon / Asset | |
| Open Issues | |
```

## DOM Verify Report

```markdown
# Figma DOM Verify Report

## Baseline
- Figma node:
- Browser URL:
- Date:
- Mode:

## DOM Diff
| Element | Figma | DOM | Diff | Severity | Fix |
|---|---:|---:|---:|---|---|

## Adjacent Box Boundary Diff
| Relation | Figma | DOM | Diff | Source CSS | Severity | Fix |
|---|---:|---:|---:|---|---|---|

## State Verification
| State | Figma Node | Browser Path / Trigger | Result | Notes |
|---|---|---|---|---|

## Remaining Deviations
| Item | Reason | Accepted? |
|---|---|---|

## Test Results
- 
```

## Subagent Handoff

```markdown
Task:
Figma nodes:
Dimension Spec:
State Matrix:
Allowed files:
Do not modify:
Expected output:
```

## Final Answer Checklist

```markdown
- Blueprint coverage:
- Changed files:
- UI states covered:
- DOM Diff Summary:
  - Critical:
  - Major:
  - Minor:
- Adjacent Box Boundary Diff Summary:
- Unverified visual items:
- Tests/type-check:
- Remaining deviations:
```
