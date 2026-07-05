# Icon & Asset Rules

用于保留并强化旧 skill 的 Icon Inventory 和图片资源决策能力。

## Project Icon System

- 包：`@your-org/icons`
- 常见版本：v2.x
- Props：`size`、`className?`、`color?`

## Figma → Code Mapping

| Figma naming | Code package | Rule |
|---|---|---|
| `01_im/{name}_1` | `@your-org/icons` | `Im{PascalCase(name)}1` |
| `01_im/{name}_2` | `@your-org/icons` | `Im{PascalCase(name)}2` |

示例：`01_im/statusTips_1` → `ImStatusTips1`。

## Existence Check

```bash
ls node_modules/.pnpm/@your-org+icons*/node_modules/@your-org/icons/dist/icons/{ComponentName}.js 2>/dev/null
rg "{ComponentName}" node_modules/.pnpm/@your-org+icons*/node_modules/@your-org/icons -g '*.d.ts' -g '*.js'
```

## Icon Verification

DOM Verify 必须检查：

- 是否为 SVG/Icon 组件。
- 是否存在字符占位：`/[←›＋×▶]/`。
- size、color、stroke/fill 是否匹配。
- icon 与文字 gap 是否匹配。

## Non-icon Assets

非 icon 图片包括：空态插画、背景图、纹理、品牌图、复杂渐变、复杂 vector illustration。

策略：

1. 优先复用项目已有 asset。
2. 若 Figma 有独立图片，导出或请求用户提供。
3. 复杂 illustration / vector asset 默认必须 export asset 或复用 design-system asset。
4. CSS recreation 仅允许用于简单几何图形；复杂资产用 CSS recreate 时必须明确标注为 accepted visual deviation。
5. 不得声称 CSS recreated complex illustration 是 1:1 fidelity。
6. 暗黑主题需检查 `body[data-color-scheme="dark"]` 或项目主题机制。

Asset strategy 只能使用：

- reuse existing asset
- export Figma asset
- accepted CSS recreation
- defer with deviation

## Asset Inventory Template

| Node | Asset | Size | Theme | Strategy | Verify Method | Deviation |
|---|---|---:|---|---|---|
