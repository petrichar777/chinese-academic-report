# Common Errors & Fixes

## Error 1: Chinese Quotes Cause SyntaxError

**Symptom:**
```
SyntaxError: missing ) after argument list
    at line XX: p("...文字中的"引用内容"引起错误...", { indent: true })
```

**Root cause:** Chinese curly double quotes (`""` / U+201C U+201D) inside `"..."` JS strings look like ASCII `"` (U+0022), which prematurely terminates the JavaScript string.

**Fix:** Use backtick template literals for ALL `p()` body text calls:
```javascript
// WRONG
p("Text with "quotes" inside", { indent: true })

// CORRECT
p(`Text with "quotes" inside`, { indent: true })
```

## Error 2: First-Line Indent Too Small

**Symptom:** Body text indentation looks like ~1.5 characters instead of 2 characters.

**Root cause:** Using 480 DXA for firstLine indent (mathematically 2 × 12pt × 20 = 480) is visually insufficient in Word rendering.

**Fix:** Always use **640** DXA for body text:
```javascript
indent: opts.indent ? { firstLine: 640 } : undefined  // NOT 480!
```

## Error 3: Module Not Found: docx

**Symptom:** `Error: Cannot find module 'docx'`

**Fix:** Install locally in the working directory:
```bash
npm install docx
```

## Error 4: Table Renders with Black Backgrounds

**Symptom:** Table cells with shading appear as solid black.

**Fix:** Use `ShadingType.CLEAR` not `ShadingType.SOLID`:
```javascript
shading: { fill: "D9E2F3", type: ShadingType.CLEAR }
```

## Error 5: Tables Render Incorrectly

**Symptom:** Table columns have wrong widths or tables overflow page margins.

**Fix checklist:**
- [ ] Use `WidthType.DXA` (not PERCENTAGE) — PERCENTAGE breaks in Google Docs
- [ ] Table `width.size` = CONTENT_W (9938 for 25mm margins)
- [ ] `columnWidths` array sums EXACTLY to table width
- [ ] Each cell has matching `width: { size: X, type: WidthType.DXA }`
- [ ] Cell `margins` are set (not undefined)

## Error 6: Document Doesn't Open in Word

**Symptom:** Word says file is corrupted.

**Likely causes:**
1. Missing `type` on `ImageRun`: always specify `type: "png"` or `type: "jpg"`
2. PageBreak outside of Paragraph: always wrap in `new Paragraph({ children: [new PageBreak()] })`
3. Duplicate bookmark IDs in internal hyperlinks

## Error 7: `\n` in text doesn't create new lines

**Symptom:** Manual `\n` in text shows as literal characters, not line breaks.

**Fix:** Never use `\n` in docx-js. Always use separate `Paragraph` elements for each line.

## Error 8: Cover Table Cell Borders Don't Match Template

**Symptom:** Cover table looks different from template (extra borders, wrong alignment).

**Root cause:** The specification requires bottom-border-only on value cells (underline style), not full borders on all cells.

**Fix:** Cover page table cells should have:
- Label cells: NO borders (`borders: {}`)
- Value cells: bottom border only (`borders: { bottom: { style: BorderStyle.SINGLE, size: 4, color: "auto" } }`)
- Table itself: center aligned

## Error 9: Wrong Page Margins

**Symptom:** Content area looks too wide or too narrow compared to specification.

**Fix:** Per 成都东软学院定制班课程报告撰写规范, use **25mm margins**:
```javascript
const MARGIN = 984;    // 25mm — NOT 1418 (~36mm)
const H_MARGIN = 567;   // 1.5cm header distance
const F_MARGIN = 661;   // 1.75cm footer distance
// Content width = 11906 - 2*984 = 9938
```

## Error 10: Wrong Header Text

**Symptom:** Header shows wrong text like "期末总结报告" instead of "定制班课程报告".

**Fix:** The specification requires header text to be exactly:
```javascript
"成都东软学院定制班课程报告"
```
Not "COURSE_NAME期末总结报告" or any other variant. The header text is fixed per the specification.

## Error 11: Wrong Cover Title

**Symptom:** Cover page shows "期末总结报告" instead of "定制班课程报告".

**Fix:** The specification requires the cover report title to be:
```javascript
"定制班课程报告"
```

## Error 12: Cover Table Uses Wrong Labels

**Symptom:** Cover table has wrong last row label.

**Fix:** The specification's cover includes "提交日期", not "实训期":
```javascript
mkRow("提交日期：", "<<提交日期>>"),
```

## Error 13: H1 Font Size Too Small

**Symptom:** Chapter titles look smaller than the specification.

**Fix:** Use size 36 (小二 18pt):
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

## Error 14: TOC Fonts Wrong

**Symptom:** TOC entries use wrong fonts — all 宋体 instead of L1 黑体.

**Fix:** The specification requires:
- L1 (章) entries: **小四号黑体**
- L2 (节) entries: 小四号宋体
- L3 (条) entries: 小四号宋体, 左缩进2字符
- Page numbers: 小四号 Times New Roman
- All: 1.5倍行距

The docx-js `TableOfContents` field auto-generates TOC entries from heading styles, so font inheritance depends on how headings are styled. Ensure heading styles use the correct fonts.

## Error 15: Abstract Format Wrong

**Symptom:** Abstract title missing 2 spaces between characters.

**Fix:** Use `"摘  要"` (2 spaces between the two characters). Same for `"目  录"` title.

## Error 16: Code Block Line Spacing Wrong

**Symptom:** Code blocks use default line spacing instead of 1.25倍.

**Fix:** Use `line: 300` and `before: 120`:
```javascript
new Paragraph({
  spacing: { before: 120, line: 300 },
  children: [new TextRun({ text: codeLine, font: "Times New Roman", size: 21 })],
})
```

## Error 17: Header Missing Bottom Border

**Symptom:** Header doesn't have the single-line bottom border required by the specification.

**Fix:** Add a bottom border to the header paragraph:
```javascript
new Paragraph({
  alignment: AlignmentType.CENTER,
  border: { bottom: { style: BorderStyle.SINGLE, size: 6, color: "auto", space: 1 } },
  children: [new TextRun({ text: "成都东软学院定制班课程报告", font: FONT_BODY, size: 21 })],
})
```

## Verification Checklist

Before delivering the final .docx, verify:
- [ ] File size is 50-70KB (for ~35-page report)
- [ ] File starts with PK bytes (valid ZIP)
- [ ] Body text uses backtick template literals (no Chinese quote syntax errors)
- [ ] `firstLine` indent is 640 (not 480)
- [ ] All tables use DXA widths and CLEAR shading
- [ ] Margins are 984 DXA (25mm), not 1418
- [ ] Header distance: 567 DXA (1.5cm)
- [ ] Footer distance: 661 DXA (1.75cm)
- [ ] Content width: 9938 DXA
- [ ] Header text: "成都东软学院定制班课程报告" (宋体 五号 size 21, centered, bottom border)
- [ ] Cover title: "定制班课程报告" (黑体 size 48)
- [ ] Cover school name: "成都东软学院" (黑体 size 52 bold)
- [ ] Cover table labels: 黑体 size 32 (三号), value cells: 宋体 size 32, bottom border only
- [ ] Cover table last row: "提交日期：" (not "实训期：")
- [ ] Headers/footers present on front matter and body sections (not cover)
- [ ] Front matter footer: 宋体 小五 size 18, Roman numerals
- [ ] Body footer: Times New Roman 小五 size 18, Arabic numerals restarted from 1
- [ ] H1: size 36 (小二), 黑体, bold, CENTERED, pageBreakBefore
- [ ] H2: size 30 (小三), 黑体, bold, 居左
- [ ] H3: size 28 (四号), 黑体, bold, 居左
- [ ] H4: size 24 (小四), 宋体, normal, 首行缩进2字符
- [ ] Body text: size 24 (小四), 宋体+Times New Roman, 1.5倍行距(line 360), JUSTIFIED
- [ ] Table cell: size 21 (五号), 宋体, center aligned
- [ ] Table caption: ABOVE table, size 21 (五号), 宋体, centered
- [ ] Figure caption: BELOW figure, size 21 (五号), 宋体, centered
- [ ] Code: size 21 (五号), Times New Roman+宋体, line 300, before 120
- [ ] Abstract title: "摘  要" (2 spaces between chars), 黑体 size 36 bold
- [ ] TOC title: "目  录" (2 spaces between chars), 黑体 size 36 bold
- [ ] Appendix has both glossary table and grading table
