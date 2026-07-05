# Intake — Whole Design & State Coverage

用于实现前理解完整视觉稿、业务范围和所有 UI 状态。Full/Standard 任务必须先读本文件。

## Inputs

- Figma root URL / nodeId
- 用户目标与执行范围
- 目标端：PC / Mobile / H5 / 多端
- 现有代码范围与约束

## Actions

1. 解析 Figma URL，记录 file、page、root node、用户指定 node。
2. 识别该 node 是页面、section、frame、component instance 还是局部 layer。
3. 扫描 root 下所有直接子层和关键 descendants，按页面主体、配置区、弹窗、popover、tooltip、空态、数据态、hover/selected/disabled、移动端等分类。
4. 做 Parent-Child Ownership Check：判断目标节点属于哪个父 frame；不得按业务字段擅自拆 UI。
5. 标注每个状态是否在本次范围内：implement / readonly / verify only / out of scope。
6. 若用户只评论局部节点，但原始请求是完整实现，必须回到 root 覆盖矩阵，不得只修局部。

## Complexity Routing

- Simple：单 icon/button/text 修复，可走 node-level extract + verify。
- Standard：单组件多状态，必须 State Coverage Matrix。
- Full：section/page/root、多弹窗、多端、多交互，必须 Whole Canvas Layer Coverage Matrix。

## Layer Coverage Matrix Template

| Node ID | Name | Layer Type | Parent Frame | Scenario / State | Target App | Implement? | Code Area | Notes |
|---|---|---|---|---|---|---|---|---|

## State Coverage Matrix Template

| Component | State | Figma Node | Trigger | Expected UI | Data Source | Implemented By | Verify Target |
|---|---|---|---|---|---|---|---|

## Scope Decision Template

```markdown
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

## Tooltip / Popover Scan

遇到 `statusTips`、`question`、`info`、`help`、`hover`、`popover`、`tooltip` 语义时：

1. 查找同组或相邻 layer 是否存在 hover/tooltip frame。
2. 记录触发元素、内容、位置、宽高、箭头、阴影、z-index。
3. 若找不到 tooltip layer，写入 Open Questions 或 Remaining Deviations，不得默默忽略。

## Output Gate

进入 Extract 前必须产出：

- Layer Coverage Matrix
- State Coverage Matrix
- Scope Decision
- Open Questions / Degraded assumptions
