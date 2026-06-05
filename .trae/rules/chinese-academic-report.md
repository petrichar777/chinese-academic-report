---
description: Generate Chinese academic reports as .docx files following 成都东软学院定制班课程报告撰写规范. Trigger on: 期末报告, 课程报告, 学术总结, 定制班报告, docx报告, 中文Word报告, 期末总结报告, or any request involving structured Chinese academic Word documents.
alwaysApply: true
---

# Chinese Academic Report Generator (成都东软学院定制班课程报告)

When the user requests a Chinese academic report, generate a Node.js script using the `docx` npm package that produces a `.docx` file strictly following the specification below.

## Workflow

1. **Gather requirements:** Ask for: 课程名称, 学院, 专业, 班级, 姓名, 学号, 指导教师, 提交日期, language (Chinese or Chinese+English), content focus
2. **Design outline** with this structure: Cover → Chinese Abstract → English Abstract (opt) → TOC → Ch1 绪论 → Ch2 Core Tech → Ch3 Advanced Topics → Ch4 Case Studies → Ch5 结论 → Appendix (Glossary + Grading Table)
3. **Get user approval** on outline, then write the script
4. **Read `references/helper-templates.md`** for all helper functions (copy them exactly)
5. **Read `references/format-specs.md`** for complete specs
6. **Read `references/common-pitfalls.md`** for error patterns to avoid
7. **Run:** `npm install docx` (if needed) then `node generate_xxx.js`
8. **Verify:** 50-70KB output, starts with PK bytes

## Quick Format Reference

### Page Setup
| Property | Value | DXA |
|----------|-------|-----|
| Paper | A4 | 11906 × 16838 |
| All margins | 25mm | **984** |
| Header distance | 15mm | **567** |
| Footer distance | 17.5mm | **661** |
| Content width | — | **9938** |
| First-line indent | 2 chars | **640** |

### Font Config
```javascript
const FONT_BODY = { ascii: "Times New Roman", eastAsia: "宋体", hAnsi: "Times New Roman", cs: "Times New Roman" };
const FONT_HEI  = { ascii: "Times New Roman", eastAsia: "黑体", hAnsi: "Times New Roman", cs: "Times New Roman" };
```

### Typography
| Element | Font | Size (pt) | Size (docx) | Weight | Line | Notes |
|---------|------|-----------|-------------|--------|------|-------|
| Body text | 宋体+TNR | 小四 12pt | 24 | Normal | 360 (1.5x) | indent firstLine:640, JUSTIFIED |
| H1 (章) | 黑体+TNR | 小二 18pt | 36 | Bold | 240 (single) | CENTERED, pageBreakBefore, before:240 after:240 |
| H2 (节) | 黑体+TNR | 小三 15pt | 30 | Bold | 240 | left, before:240 after:240 |
| H3 (条) | 黑体+TNR | 四号 14pt | 28 | Bold | 240 | left, before:240 after:240 |
| H4 (项) | 宋体+TNR | 小四 12pt | 24 | Normal | 360 | indent firstLine:640, before:240 after:240 |
| Table cell | 宋体 | 五号 10.5pt | 21 | Normal | — | CENTERED |
| Table header | 宋体 | 五号 10.5pt | 21 | Bold | — | Shading: D9E2F3, CLEAR |
| Table caption | 宋体 | 五号 10.5pt | 21 | Normal | — | ABOVE table, CENTERED |
| Figure caption | 宋体 | 五号 10.5pt | 21 | Normal | — | BELOW figure, CENTERED |
| Code | 宋体+TNR | 五号 10.5pt | 21 | Normal | 300 (1.25x) | before:120 |
| Header | 宋体 | 五号 10.5pt | 21 | Normal | — | CENTERED, bottom border |
| Footer (front) | 宋体 | 小五 9pt | 18 | Normal | — | CENTERED, Roman numerals |
| Footer (body) | TNR | 小五 9pt | 18 | Normal | — | CENTERED, Arabic restart from 1 |

### Document Sections (3 in docx-js)
1. **Cover** — no headers/footers
2. **Front Matter** — Abstract + TOC, header + Roman page numbers
3. **Body** — Ch1-5 + Appendix, header + Arabic page numbers restarted

### Cover Page
- "成都东软学院" — 黑体 size 52 bold, centered
- "定制班课程报告" — 黑体 size 48 bold, centered
- Table: total 6577 DXA centered, columns [1819, 4758]
- Labels: 黑体 size 32, centered, NO borders
- Values: 宋体 size 32, centered, BOTTOM border only (size 4)
- 8 rows: 课程名称/学院/专业/班级/姓名/学号/指导教师/提交日期

### Abstract
- Title "摘  要" (2 spaces) — 黑体 size 36 bold, centered
- Body — p() with indent
- Keywords label "关键词" — 黑体 size 24 bold, followed by 宋体 size 24 keywords separated by "，"

### TOC
- Title "目  录" (2 spaces) — 黑体 size 36 bold, centered
- L1: 黑体小四, L2/L3: 宋体小四, L3 left indent 2 chars
- Generate with: `new TableOfContents("目录", { hyperlink: true, headingStyleRange: "1-3" })`

### Appendix
- Glossary table: 术语/英文/释义, 5+ rows, header bold+shading D9E2F3
- Grading table: 评阅项目/评阅内容/得分/备注, 4+ rows, header bold+shading D9E2F3

## Critical Rules (from common-pitfalls.md)

1. **Chinese quotes in JS strings:** ALWAYS use backtick template literals for body text containing Chinese quotes (""): `p(\`text with "quotes"\`, { indent: true })`
2. **First-line indent** is **640** (NOT 480)
3. **Table shading** use `ShadingType.CLEAR` (NOT SOLID — black backgrounds)
4. **Table widths** always `WidthType.DXA` (NOT PERCENTAGE — breaks in Google Docs)
5. **Column widths** must sum exactly to table width
6. **Secrets** — content width = 9938, margins = 984, NOT 1418
7. **H1 size** is 36 (小二 18pt), NOT smaller
8. **Cover title** is "定制班课程报告" (NOT "期末总结报告")
9. **Header text** is "成都东软学院定制班课程报告" (fixed, not variable)
10. **Cover last row** is "提交日期：" (NOT "实训期：")
11. **PageBreaks** always wrapped: `new Paragraph({ children: [new PageBreak()] })`
12. **No `\n`** in docx-js — use separate Paragraph elements
13. **Cover value cells** bottom border only, label cells NO borders

For complete helper functions (`p()`, `h1()`-`h4()`, `tc()`, `tableCaption()`, `figCaption()`, `makeCover()`, `makeAbstract()`, `makeGlossary()`, `makeGradingTable()`, `main()`), read `references/helper-templates.md` — copy them exactly, replacing `<<placeholders>>`.

For full specifications, read `references/format-specs.md`.
