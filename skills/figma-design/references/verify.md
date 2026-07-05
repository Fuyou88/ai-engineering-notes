# Verify — Browser DOM Diff

用于验证实现是否还原 Figma。无 DOM Diff Report 不得声称完成。

## Inputs

- Dimension Spec / Blueprint
- Browser URL
- 关键 DOM selector / accessible target
- 状态触发方式

## Measurement Method

使用浏览器 DevTools 执行 `getBoundingClientRect()`，不要用 `getComputedStyle()` 推测尺寸。

```js
(selector) => {
  const element = document.querySelector(selector);
  if (!element) return { found: false };
  const rect = element.getBoundingClientRect();
  const style = getComputedStyle(element);
  return {
    found: true,
    x: rect.x,
    y: rect.y,
    width: rect.width,
    height: rect.height,
    paddingTop: style.paddingTop,
    paddingRight: style.paddingRight,
    paddingBottom: style.paddingBottom,
    paddingLeft: style.paddingLeft,
    marginTop: style.marginTop,
    marginRight: style.marginRight,
    marginBottom: style.marginBottom,
    marginLeft: style.marginLeft,
    borderRadius: style.borderRadius,
    background: style.backgroundColor,
    color: style.color,
    fontSize: style.fontSize,
    fontWeight: style.fontWeight,
    lineHeight: style.lineHeight,
    display: style.display,
    gap: style.gap,
  };
}
```

## Screenshot Preflight

截图对比是低成本视觉预筛，不是完成标准。

- 当用户提供截图、框选区域或视觉偏差反馈时，先用截图定位明显错误目标。
- 截图明显不符时，不得声明完成；必须对明显错误目标执行 DOM Diff。
- 截图用于发现问题，DOM Diff 用于定位原因并验证修复。
- type-check、单测、构建成功不得作为视觉验收。

## User-Pointed Target Rule

如果用户提供具体 selector、截图框选、可见文案或明确元素描述，该目标必须纳入 DOM Diff。

- 不得用父元素、相邻元素或其它状态替代用户指出的目标。
- 如果目标无法找到，必须明确说明，并从当前可见 DOM 重新定位。

## Third-party Internal DOM Diff

使用 UI library 组件实现的目标，必须同时测业务 wrapper 和真实内部 DOM。

| Component | Required inner DOM |
|---|---|
| Input / InputNumber | wrapper、actual input、prefix/suffix、stepper controls |
| Select | root、selector、selection item、arrow、dropdown trigger |
| Checkbox / Radio | wrapper、inner box、label text、adjacent help/info icon |
| DatePicker | input wrapper、actual input、suffix icon、popup trigger |

当 Figma 指定控件盒模型、文字、图标、边框或相邻间距时，只测业务 wrapper 不算完成。

## DOM Verify Payload Economy

测量 DOM 时只返回 diff 所需字段，不 dump 全量 computed styles。

- 必需字段：x/y/width/height、padding/margin、border、font-size/line-height/color。
- 仅相关时返回 display/gap/background/borderRadius。
- 按组件组测量，不测整个 document。
- 重复列表行只测第一行和一个代表行。
- 输出 summary diff，并展开 Critical/Major 细节。

## DOM Diff Report Template

```markdown
# Figma DOM Verify Report

## Baseline
- Figma node:
- Browser URL:
- Date:
- Mode: full / standard / simple

## DOM Diff
| Element | Figma | DOM | Diff | Severity | Fix |
|---|---:|---:|---:|---|---|

## State Verification
| State | Figma Node | Browser Path / Trigger | Result | Notes |
|---|---|---|---|---|

## Remaining Deviations
| Item | Reason | Accepted? |
|---|---|---|
```

## Adjacent Box Boundary Diff
Use this when verifying spacing, gap, padding, alignment, or visual adjacency. CSS gap/padding values alone are not sufficient.

| Relation | Figma | DOM | Diff | Source CSS | Severity | Fix |
|---|---:|---:|---:|---|---|---|
| parent.left → firstChild.left | 8 | 8 | 0 | padding-left | Pass | - |
| childA.right → childB.left | 8 | 28 | +20 | wrapper margin-right | Critical | override wrapper margin |

Adjacent gap helper snippet stays inline here. Extract it into a reusable script only if it grows beyond ~80 lines or is reused across multiple verification workflows.

```js
function horizontalGap(leftEl, rightEl) {
  const left = leftEl.getBoundingClientRect();
  const right = rightEl.getBoundingClientRect();
  return right.left - left.right;
}

function verticalGap(topEl, bottomEl) {
  const top = topEl.getBoundingClientRect();
  const bottom = bottomEl.getBoundingClientRect();
  return bottom.top - top.bottom;
}
```

## Severity Rules

| Severity | Definition | Required Action |
|---|---|---|
| Critical | diff > 5px、布局结构错、icon 缺失/字符占位、状态缺失、数据不一致 | 必须修复 |
| Major | diff 3–5px、颜色/字重/按钮/tooltip 明显不一致 | 修复或说明不可行原因 |
| Minor | diff ≤ 2px、亚像素或平台渲染差异 | 可接受但需记录 |

## Checklist

- 尺寸
- 间距
- padding / margin
- 相邻盒模型边界（spacing / gap / padding / alignment / visual adjacency）
- header / body / footer 分割线
- icon / asset
- typography / line-height
- third-party default DOM
- interaction state
- empty / loading / error
- dark mode / mobile if in scope
- data consistency

## Fix Loop

1. 按 Critical → Major → Minor 排序。
2. 每次只修一个明确 diff 或同源 diff 组。
3. 修复后立即重新测量对应 DOM。
4. 不得批量靠猜修样式。
5. Critical 清零后才可进入最终汇报。

## Final Verification Output

最终必须输出：

- Diff summary：Critical / Major / Minor 数量。
- 修复清单。
- 状态覆盖结果。
- 剩余偏差和是否接受。
- 测试/type-check 结果。

如果无法执行 DOM measurement，必须明确说明：

- 无法测量的原因。
- 使用了哪个 baseline 替代。
- 哪些视觉项仍未验证。

没有 DOM measurement 时，不得声称 pixel-perfect completion。
