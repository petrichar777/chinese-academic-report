# Format Specifications — 成都东软学院定制班课程报告撰写规范

Complete format specifications extracted from the authoritative `报告撰写规范.md`. All values below are the **default** standard; override only when the user provides explicit alternative requirements.

---

## 1. Page Setup (页面设置)

| Property | Value | DXA | Notes |
|----------|-------|-----|-------|
| Paper size | A4 (210mm × 297mm) | 11906 × 16838 | |
| Top margin | 25mm | **984** | |
| Bottom margin | 25mm | **984** | |
| Left margin | 25mm | **984** | |
| Right margin | 25mm | **984** | |
| Header distance | 15mm (1.5cm) | **567** | Distance from top edge of page |
| Footer distance | 17.5mm (1.75cm) | **661** | Distance from bottom edge of page |
| Content width | — | **9938** | 11906 - 2×984 |

Page numbering:
- Front matter (摘要, 目录): uppercase Roman numerals (I, II, III…), 宋体小五号, centered
- Body (正文起): Arabic numerals (1, 2, 3…), Times New Roman 小五号, centered, restart from 1

---

## 2. Default Font Configuration

```javascript
const FONT_BODY = { ascii: "Times New Roman", eastAsia: "宋体", hAnsi: "Times New Roman", cs: "Times New Roman" };
const FONT_HEI  = { ascii: "Times New Roman", eastAsia: "黑体", hAnsi: "Times New Roman", cs: "Times New Roman" };
```

Rule: Chinese text uses the `eastAsia` font; digits and English use the `ascii`/`hAnsi` font.

---

## 3. Body Text (正文)

| Property | Value | docx-js |
|----------|-------|---------|
| Font (中文) | 宋体 | `eastAsia: "宋体"` |
| Font (数字/英文) | Times New Roman | `ascii: "Times New Roman"` |
| Size | 小四 12pt | `size: 24` |
| Line spacing | 1.5倍行距 | `line: 360` |
| First-line indent | 2个中文字符 | `firstLine: 640` |
| Alignment | 两端对齐 (justified) | `AlignmentType.JUSTIFIED` |

```javascript
function p(text, opts = {}) {
  return new Paragraph({
    spacing: { line: 360 },
    indent: opts.indent ? { firstLine: 640 } : undefined,
    alignment: opts.align || AlignmentType.JUSTIFIED,
    children: [new TextRun({ text, font: FONT_BODY, size: 24 })],
  });
}
```

---

## 4. Headings (标题)

### H1 — Chapter Titles (一级标题 / 第X章)

| Property | Value | docx-js |
|----------|-------|---------|
| Font (中文) | 黑体 | `FONT_HEI` |
| Font (数字/英文) | Times New Roman | via `FONT_HEI` |
| Size | 小二 18pt | `size: 36` |
| Bold | Yes | `bold: true` |
| Alignment | 居中 | `AlignmentType.CENTER` |
| Spacing before | 1行 (~240) | `before: 240` |
| Spacing after | 1行 (~240) | `after: 240` |
| Line spacing | 单倍行距 | `line: 240` |
| Page break | 另起一页 | `pageBreakBefore: true` |

Format: `第X章　标题` (Chinese full-width space U+3000 between chapter number and title)

```javascript
function h1(text) {
  return new Paragraph({
    heading: HeadingLevel.HEADING_1,
    spacing: { before: 240, after: 240 },
    alignment: AlignmentType.CENTER,
    pageBreakBefore: true,
    children: [new TextRun({ text, font: FONT_HEI, size: 36, bold: true })],
  });
}
```

### H2 — Section Headings (二级标题)

| Property | Value | docx-js |
|----------|-------|---------|
| Font (中文) | 黑体 | `FONT_HEI` |
| Font (数字/英文) | Times New Roman | via `FONT_HEI` |
| Size | 小三 15pt | `size: 30` |
| Bold | Yes | `bold: true` |
| Alignment | 居左 | (default) |
| Spacing before | 1行 (~240) | `before: 240` |
| Spacing after | 1行 (~240) | `after: 240` |
| Line spacing | 单倍行距 | `line: 240` |

Format: `X.X XXXXXX`

```javascript
function h2(text) {
  return new Paragraph({
    heading: HeadingLevel.HEADING_2,
    spacing: { before: 240, after: 240 },
    children: [new TextRun({ text, font: FONT_HEI, size: 30, bold: true })],
  });
}
```

### H3 — Sub-section Headings (三级标题)

| Property | Value | docx-js |
|----------|-------|---------|
| Font (中文) | 黑体 | `FONT_HEI` |
| Font (数字/英文) | Times New Roman | via `FONT_HEI` |
| Size | 四号 14pt | `size: 28` |
| Bold | Yes | `bold: true` |
| Alignment | 居左 | (default) |
| Spacing before | 1行 (~240) | `before: 240` |
| Spacing after | 1行 (~240) | `after: 240` |
| Line spacing | 单倍行距 | `line: 240` |

Format: `X.X.X XXXXXX`

```javascript
function h3(text) {
  return new Paragraph({
    heading: HeadingLevel.HEADING_3,
    spacing: { before: 240, after: 240 },
    children: [new TextRun({ text, font: FONT_HEI, size: 28, bold: true })],
  });
}
```

### H4 — Item Headings (四级标题)

| Property | Value | docx-js |
|----------|-------|---------|
| Font (中文) | 宋体 | `FONT_BODY` |
| Font (数字/英文) | Times New Roman | via `FONT_BODY` |
| Size | 小四 12pt | `size: 24` |
| Bold | Normal | (no bold) |
| First-line indent | 2个字符 | `firstLine: 640` |
| Spacing before | 1行 (~240) | `before: 240` |
| Spacing after | 1行 (~240) | `after: 240` |
| Line spacing | 1.5倍行距 | `line: 360` |

Uses alternative numbering: (a)(b)(c)… or (1)(2)(3)… etc.

---

## 5. Cover Page (封面)

### School Name
- Text: "成都东软学院"
- Font: 黑体, size **52** (一号 26pt), **bold**
- Alignment: CENTERED

### Report Title
- Text: "定制班课程报告"
- Font: 黑体, size **48**, bold
- Alignment: CENTERED

### Course Info Table
- Table width: centered, total ~**6577 DXA**
- Column widths: **[1819, 4758]** (label + value)

**Label column (left):**
- Font: 黑体, size **32** (三号 16pt)
- Alignment: CENTERED
- No borders

**Value column (right):**
- Font: 宋体 (FONT_BODY), size **32** (三号 16pt)
- Alignment: CENTERED
- Border: **bottom only** (single, size 4, color auto) — underline effect

Labels: "课    程：", "学    院：", "专    业：", "班    级：", "学生姓名：", "学    号：", "指导教师：", "提交日期："

### Date
- Text: "<<提交日期>>" (user-provided date, e.g., "2025年6月")
- Font: 宋体 (FONT_BODY), size 24 (小四)
- Alignment: CENTERED

---

## 6. Header & Footer (页眉 & 页脚)

### Header (自摘要页起)

| Property | Value |
|----------|-------|
| Text | "成都东软学院定制班课程报告" |
| Font | 宋体 (FONT_BODY) |
| Size | 五号 10.5pt (`size: 21`) |
| Alignment | CENTERED |
| Bottom border | Single line |

```javascript
headers: {
  default: new Header({
    children: [new Paragraph({
      alignment: AlignmentType.CENTER,
      border: { bottom: { style: BorderStyle.SINGLE, size: 6, color: "auto", space: 1 } },
      children: [new TextRun({ text: "成都东软学院定制班课程报告", font: FONT_BODY, size: 21 })],
    })],
  }),
}
```

### Footer

| Section | Style | Font | Size |
|---------|-------|------|------|
| Front matter (摘要, 目录) | 大写罗马数字 | 宋体 | 小五 9pt (`size: 18`) |
| Body (正文起) | 阿拉伯数字, restart from 1 | Times New Roman | 小五 9pt (`size: 18`) |

Both centered. Use `PageNumber.CURRENT`.

---

## 7. Abstract (摘要)

### Chinese Abstract (中文摘要)

| Element | Font | Size | Format |
|---------|------|------|--------|
| Title "摘  要" | 黑体 | 小二 18pt (`size: 36`) | 居中, 二字间2空格, 单倍行距 |
| Body text | 宋体 | 小四 12pt (`size: 24`) | 首行缩进2字符, 1.5倍行距 |
| Label "关键词" | 黑体 | 小四 12pt (`size: 24`) | **加粗** |
| Keywords | 宋体 | 小四 12pt (`size: 24`) | 词间用"，"分隔, 最后一个关键词无标点 |

Body text and keywords separated by one empty line.

### English Abstract (英文摘要, optional)

| Element | Font | Size | Format |
|---------|------|------|--------|
| Title "Abstract" | Times New Roman | 小二 18pt (`size: 36`) | 居中, 单倍行距 |
| Body text | Times New Roman | 小四 12pt (`size: 24`) | 首行缩进2字符, 1.5倍行距 |
| Label "Key Words" | 黑体 | 小四 12pt (`size: 24`) | **加粗** |
| Keywords | Times New Roman | 小四 12pt (`size: 24`) | 词间用英文半角","分隔 |

---

## 8. Table of Contents (目录)

### TOC Title
- Text: "目  录" (二字间2空格)
- Font: 黑体, size **36** (小二 18pt), bold
- Alignment: CENTERED

### TOC Entries

| Level | Font (中文) | Font (数字/英文) | Size | Line Spacing | Special |
|-------|------------|-----------------|------|-------------|---------|
| L1 (章) | 黑体 | Times New Roman | 小四 12pt (`size: 24`) | 1.5倍行距 (`line: 360`) | Bold |
| L2 (节) | 宋体 | Times New Roman | 小四 12pt (`size: 24`) | 1.5倍行距 (`line: 360`) | |
| L3 (条) | 宋体 | Times New Roman | 小四 12pt (`size: 24`) | 1.5倍行距 (`line: 360`) | 左缩进2字符 |

- Dot leaders ("·······") between title and page number — Times New Roman 小四号
- Page numbers: Times New Roman 小四号, right-aligned
- Wrapped titles: second line indented by 1 character

### docx-js TOC:
```javascript
new TableOfContents("目录", { hyperlink: true, headingStyleRange: "1-3" })
```

---

## 9. Figures (插图)

| Element | Font | Size | Position |
|---------|------|------|----------|
| 图题 (caption) | 宋体 | 五号 10.5pt (`size: 21`) | **BELOW** figure, centered |
| 图中标注文字 | — | ≤五号 (建议五号) | — |

Format: `图 X.X 图题文字` (chapter-based numbering, one space between 图号 and 图题)

---

## 10. Tables (表格)

| Element | Font | Size | Position |
|---------|------|------|----------|
| 表题 (caption) | 宋体 | 五号 10.5pt (`size: 21`) | **ABOVE** table, centered |
| 表内文字 | 宋体 | 五号 10.5pt (`size: 21`) | — |
| 表头 (header) | 宋体 | 五号 10.5pt (`size: 21`) | Bold, shading D9E2F3 |

Format: `表 X.X 表题文字` (chapter-based numbering, one space between 表号 and 表题)

Table notes:
- "空白" = not tested / no item
- "…" = not found
- "0" = measured result is zero
- Multi-page tables: repeat header row, mark `（续表X-X）` in upper-right

```javascript
const border = { style: BorderStyle.SINGLE, size: 1, color: "000000" };
const borders = { top: border, bottom: border, left: border, right: border };

function tc(text, width, opts = {}) {
  return new TableCell({
    borders,
    width: { size: width, type: WidthType.DXA },
    margins: { top: 60, bottom: 60, left: 100, right: 100 },
    shading: opts.shading ? { fill: opts.shading, type: ShadingType.CLEAR } : undefined,
    verticalAlign: "center",
    children: [new Paragraph({
      alignment: opts.align || AlignmentType.CENTER,
      children: [new TextRun({ text, font: FONT_BODY, size: 21, bold: opts.bold || false })],
    })],
  });
}
```

---

## 11. Formulas / Equations (公式)

| Property | Value |
|----------|-------|
| Numbering | By chapter, e.g., 式(3-1) = Chapter 3, Formula 1 |
| Position | Formula centered on its own line; equation number right-aligned |

---

## 12. Code Blocks (代码)

| Property | Value | docx-js |
|----------|-------|---------|
| Font (英文) | Times New Roman | `ascii: "Times New Roman"` |
| Font (中文) | 宋体 | `eastAsia: "宋体"` |
| Size | 五号 10.5pt | `size: 21` |
| Bold | No | (no bold) |
| Line spacing | 多倍行距 1.25 | `line: 300` |
| First line | 段前 0.5行 (~120) | `before: 120` |
| Subsequent lines | 段前段后各0行 | — |

---

## 13. Citations (引用)

| Property | Value |
|----------|-------|
| Font | Times New Roman |
| Size | 小四 12pt (`size: 24`) |
| Format | Superscript (上标) |
| Numbering | By first appearance order |
| Repeated citations | Use the first occurrence number |

---

## 14. Appendix (附录)

### 术语表 (Glossary Table)
- 3 columns: 术语 / 英文 / 释义
- Column widths sum to CONTENT_W (9938)
- Header row: bold + D9E2F3 shading
- Minimum 5 terminology rows

### 指导教师评阅意见表 (Grading Table)
- Title H1: "指导教师评阅意见表"
- 4 columns: 评阅项目 / 评阅内容 / 得分 / 备注
- 4+ grading criteria rows
- Header: bold + D9E2F3 shading

---

## 15. Document Section Structure

| Section | Content | Header | Footer (Page #) |
|---------|---------|--------|-----------------|
| 1 | Cover page | none | none |
| 2 | Chinese Abstract | "成都东软学院定制班课程报告" | Roman (I, II…) |
| 3 | English Abstract (optional) | same | Roman (continued) |
| 4 | Table of Contents | same | Roman (continued) |
| 5 | Body (Ch 1-5) | same | Arabic (1, 2, 3… restarted) |
| 6 | Appendix | same | Arabic (continued) |

For docx-js, use 3 sections:
1. Cover (no headers/footers)
2. Front matter — Abstract + TOC (with headers, Roman page numbers)
3. Body — Chapters 1-5 + Appendix (with headers, Arabic page numbers restarted)

---

## 16. Font Size Mapping

| Chinese Name | Points | docx `size` (half-points) | Used For |
|-------------|--------|--------------------------|----------|
| 一号 | 26pt | 52 | Cover school name, report title |
| 小二 | 18pt | 36 | H1 (章标题), 摘要/目录 title |
| 三号 | 16pt | 32 | Cover table labels & values |
| 小三 | 15pt | 30 | H2 (节标题) |
| 四号 | 14pt | 28 | H3 (条标题) |
| 小四 | 12pt | 24 | Body text, H4, TOC entries, abstract body |
| 五号 | 10.5pt | 21 | Table cell, captions, header, code |
| 小五 | 9pt | 18 | Page numbers |

---

## 17. Line Spacing Reference

| Description | docx `line` value | Formula |
|-------------|------------------|---------|
| 单倍行距 | **240** | 240 × 1.0 |
| 多倍行距 1.25 | **300** | 240 × 1.25 |
| 1.5倍行距 | **360** | 240 × 1.5 |

---

## 18. Colors

| Element | Color Code |
|---------|-----------|
| Default text | `000000` (black) |
| Table header shading | `D9E2F3` (light blue-gray) |
| Image placeholder text | `888888` / `999999` (gray) |
| Image placeholder border | `AAAAAA` (light gray) |
