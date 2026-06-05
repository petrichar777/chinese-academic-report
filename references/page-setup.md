# Page Setup Reference — 成都东软学院定制班课程报告撰写规范

## Paper Size

| Property | Value | DXA |
|----------|-------|-----|
| Paper | A4 (210mm × 297mm) | 11906 × 16838 |

## Margins

| Margin | mm | DXA |
|--------|----|-----|
| Top (上) | 25mm | **984** |
| Bottom (下) | 25mm | **984** |
| Left (左) | 25mm | **984** |
| Right (右) | 25mm | **984** |

## Header / Footer Distances

| Distance | mm | DXA |
|----------|----|-----|
| Header from top edge (页眉距边距) | 15mm = 1.5cm | **567** |
| Footer from bottom edge (页脚距边距) | 17.5mm = 1.75cm | **661** |

## Computed Values

```javascript
const PAGE_W = 11906;   // A4 width
const PAGE_H = 16838;   // A4 height
const MARGIN = 984;      // 25mm (规范明确: 上下左右各25mm)
const H_MARGIN = 567;    // 1.5cm header distance
const F_MARGIN = 661;    // 1.75cm footer distance
const CONTENT_W = PAGE_W - 2 * MARGIN;  // 9938 DXA
```

## Common Paper Sizes (DXA)

| Paper | Width | Height | Content Width (25mm margins) |
|-------|-------|--------|---------------------------|
| **A4** (规范标准) | 11,906 | 16,838 | 9,938 |
| US Letter | 12,240 | 15,840 | 10,272 |

## Indent Reference

| Description | DXA | Notes |
|-------------|-----|-------|
| First-line indent (2 Chinese chars, 12pt 小四) | **640** | 规范: 首行缩进2个中文字符 |
| H4 first-line indent | 640 | Same as body text |
| TOC Level 3 left indent | ~480 (2 chars) | 规范: 三级标题目录左缩进2个字符 |
| Hanging indent | 360 | For bullet lists |

**Why 640 not 480?** The mathematically computed 2-char indent at 12pt = 480 DXA (12pt × 20 DXA/pt × 2), but Word rendering requires 640 DXA for correct visual appearance. Confirmed across multiple reports.

## Line Spacing Reference

| Description | docx `line` value | Notes |
|-------------|------------------|-------|
| 1.5倍行距 (正文, 目录) | **360** | `line: 360` |
| 单倍行距 (标题) | **240** | `line: 240` |
| 多倍行距 1.25 (代码) | **300** | `line: 300` |

Line spacing is set with `spacing: { line: N }` where N = 240 × multiplier.

## Table Sizing

- Always use `WidthType.DXA` — NEVER `WidthType.PERCENTAGE` (incompatible with Google Docs)
- Full-width table: `width.size` = `CONTENT_W` (9938)
- `columnWidths` array must sum exactly to table width
- Each cell must have matching `width: { size: X, type: WidthType.DXA }`
- Cell margins (internal padding): `{ top: 60, bottom: 60, left: 100, right: 100 }`

## Cover Page Table

Cover info table is narrower than content width for visual balance:
- Table total width: **6577 DXA**, centered
- Columns: **[1819, 4758]** (label + value)
- Label cells: NO borders; Value cells: bottom border only (underline effect)

## Section Structure

| Section | Content | Header | Footer (Page #) |
|---------|---------|--------|-----------------|
| 1 | Cover | none | none |
| 2 | Chinese Abstract | "成都东软学院定制班课程报告" | Roman (I, II…) |
| 3 | English Abstract | same | Roman (continued) |
| 4 | Table of Contents | same | Roman (continued) |
| 5 | Body (Ch 1-5) | same | Arabic (1, 2, 3… restarted) |
| 6 | Appendix | same | Arabic (continued) |

For simplicity in docx-js, combine sections 2-4 into one TOC section and 5-6 into one body section.
