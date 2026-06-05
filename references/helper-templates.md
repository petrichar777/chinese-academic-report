# Helper Function Templates

Copy the entire code block below at the top of every generation script. Replace all `<<...>>` placeholders with the user's actual information before running.

```javascript
const fs = require("fs");
const {
  Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell,
  Header, Footer, AlignmentType, LevelFormat,
  TableOfContents, HeadingLevel, BorderStyle, WidthType, ShadingType,
  PageNumber, PageBreak
} = require("docx");

// ============================================================
// CONSTANTS — 成都东软学院定制班课程报告撰写规范
// ============================================================
const PAGE_W = 11906, PAGE_H = 16838;   // A4
const MARGIN = 984;      // 25mm (规范: 上下左右各25mm)
const H_MARGIN = 567;    // 1.5cm header distance
const F_MARGIN = 661;    // 1.75cm footer distance
const CONTENT_W = PAGE_W - 2 * MARGIN;  // 9938 DXA

// Standard table border (thin single line)
const border = { style: BorderStyle.SINGLE, size: 1, color: "000000" };
const borders = { top: border, bottom: border, left: border, right: border };
// Cover page bottom border (thicker, for underline effect)
const coverBorder = { style: BorderStyle.SINGLE, size: 4, color: "auto" };

// Cell internal padding
const cellMargins = { top: 60, bottom: 60, left: 100, right: 100 };

// Font definitions for CJK + Latin split
const FONT_BODY = { ascii: "Times New Roman", eastAsia: "宋体", hAnsi: "Times New Roman", cs: "Times New Roman" };
const FONT_HEI  = { ascii: "Times New Roman", eastAsia: "黑体", hAnsi: "Times New Roman", cs: "Times New Roman" };

// ============================================================
// BODY PARAGRAPH helper
// Spec: 小四号(12pt) 宋体+Times New Roman, 1.5倍行距(line 360),
//       首行缩进2字符(firstLine 640)
// CRITICAL: Use backtick template literals for text containing Chinese quotes!
// ============================================================
function p(text, opts = {}) {
  const runs = [];
  if (typeof text === "string") {
    runs.push(new TextRun({ text, font: FONT_BODY, size: 24, ...opts.run }));
  } else if (Array.isArray(text)) {
    text.forEach(t => {
      if (typeof t === "string") runs.push(new TextRun({ text: t, font: FONT_BODY, size: 24, ...opts.run }));
      else runs.push(new TextRun({ font: FONT_BODY, size: 24, ...opts.run, ...t }));
    });
  }
  return new Paragraph({
    spacing: { line: 360 },
    indent: opts.indent ? { firstLine: 640 } : undefined,
    alignment: opts.align || AlignmentType.JUSTIFIED,
    ...opts.para,
    children: runs,
  });
}

// ============================================================
// HEADINGS — 黑体 + Times New Roman
// H1: 小二18pt (size 36), bold, CENTERED, pageBreakBefore, 单倍行距
//      Format: "第X章　标题" (Chinese full-width space U+3000)
// H2: 小三15pt (size 30), bold, 居左, 单倍行距
//      Format: "X.X 标题"
// H3: 四号14pt (size 28), bold, 居左, 单倍行距
//      Format: "X.X.X 标题"
// All: 段前1行(before 240), 段后1行(after 240)
// ============================================================
function h1(text) {
  return new Paragraph({
    heading: HeadingLevel.HEADING_1,
    spacing: { before: 240, after: 240 },
    alignment: AlignmentType.CENTER,
    pageBreakBefore: true,
    children: [new TextRun({ text, font: FONT_HEI, size: 36, bold: true })],
  });
}
function h2(text) {
  return new Paragraph({
    heading: HeadingLevel.HEADING_2,
    spacing: { before: 240, after: 240 },
    children: [new TextRun({ text, font: FONT_HEI, size: 30, bold: true })],
  });
}
function h3(text) {
  return new Paragraph({
    heading: HeadingLevel.HEADING_3,
    spacing: { before: 240, after: 240 },
    children: [new TextRun({ text, font: FONT_HEI, size: 28, bold: true })],
  });
}
// H4: 小四号宋体, 首行缩进2字符, 段前1行段后1行, 1.5倍行距
function h4(text) {
  return new Paragraph({
    heading: HeadingLevel.HEADING_4,
    spacing: { before: 240, after: 240, line: 360 },
    indent: { firstLine: 640 },
    children: [new TextRun({ text, font: FONT_BODY, size: 24 })],
  });
}

// ============================================================
// TABLE CELL helper
// Spec: 五号(10.5pt) 宋体+Times New Roman, center aligned
// ============================================================
function tc(text, width, opts = {}) {
  const runs = typeof text === "string"
    ? [new TextRun({ text, font: FONT_BODY, size: 21, bold: opts.bold || false })]
    : text.map(t => new TextRun({ font: FONT_BODY, size: 21, ...t }));
  return new TableCell({
    borders,
    width: { size: width, type: WidthType.DXA },
    margins: cellMargins,
    shading: opts.shading ? { fill: opts.shading, type: ShadingType.CLEAR } : undefined,
    verticalAlign: "center",
    children: [new Paragraph({ alignment: opts.align || AlignmentType.CENTER, children: runs })],
  });
}

// ============================================================
// TABLE CAPTION — placed ABOVE table
// Spec: 五号(10.5pt) 宋体, centered
// Format: "表X.X 表格标题"
// ============================================================
function tableCaption(text) {
  return new Paragraph({
    spacing: { before: 200, after: 80 },
    alignment: AlignmentType.CENTER,
    children: [new TextRun({ text, font: FONT_BODY, size: 21 })],
  });
}

// ============================================================
// FIGURE CAPTION — placed BELOW figure
// Spec: 五号(10.5pt) 宋体, centered
// Format: "图 X.X 描述文字"
// ============================================================
function figCaption(text) {
  return new Paragraph({
    spacing: { before: 80, after: 160 },
    alignment: AlignmentType.CENTER,
    children: [new TextRun({ text, font: FONT_BODY, size: 21 })],
  });
}

// ============================================================
// IMAGE PLACEHOLDER (dashed border — for reports without real images)
// ============================================================
function imgPlaceholder(desc, figNum = "") {
  const label = figNum ? `【图${figNum}】` : "【图片位置】";
  return [
    new Paragraph({ spacing: { before: 200, after: 80 }, alignment: AlignmentType.CENTER,
      children: [new TextRun({ text: `[ ${label} ${desc} ]`, font: FONT_BODY, size: 21, italics: true, color: "888888" })] }),
    new Paragraph({ spacing: { before: 40, after: 60 },
      border: { top: { style: BorderStyle.DASHED, size: 2, color: "AAAAAA" }, bottom: { style: BorderStyle.DASHED, size: 2, color: "AAAAAA" }, left: { style: BorderStyle.DASHED, size: 2, color: "AAAAAA" }, right: { style: BorderStyle.DASHED, size: 2, color: "AAAAAA" } },
      alignment: AlignmentType.CENTER,
      children: [new TextRun({ text: `（此处插入图片：${desc}）`, font: FONT_BODY, size: 20, color: "999999" })] }),
    figCaption(figNum ? `图 ${figNum}` : ""),
  ];
}

// ============================================================
// COVER PAGE TABLE ROW
// Label: 黑体 三号(size 32), centered, no borders
// Value: 宋体 三号(size 32), centered, bottom border only (underline)
// ============================================================
function mkRow(label, value) {
  return new TableRow({ children: [
    new TableCell({
      borders: {},
      width: { size: 1819, type: WidthType.DXA },
      margins: cellMargins,
      verticalAlign: "bottom",
      children: [new Paragraph({
        alignment: AlignmentType.CENTER,
        children: [new TextRun({ text: label, font: FONT_HEI, size: 32 })],
      })],
    }),
    new TableCell({
      borders: { bottom: coverBorder },
      width: { size: 4758, type: WidthType.DXA },
      margins: cellMargins,
      verticalAlign: "bottom",
      children: [new Paragraph({
        alignment: AlignmentType.CENTER,
        children: [new TextRun({ text: value, font: FONT_BODY, size: 32 })],
      })],
    }),
  ]});
}

// ============================================================
// COVER PAGE
// School name: 黑体 size 52 (一号) bold, centered
// Report title: 黑体 size 48 bold, centered
// Info table: 8 rows, total width 6577 DXA, centered
// ============================================================
function makeCover() {
  return [
    new Paragraph({ spacing: { after: 100 }, alignment: AlignmentType.CENTER,
      children: [new TextRun({ text: "", font: FONT_HEI, size: 48 })] }),
    new Paragraph({ spacing: { line: 360 }, alignment: AlignmentType.CENTER,
      children: [new TextRun({ text: "成都东软学院", font: FONT_HEI, size: 52, bold: true })] }),
    new Paragraph({ spacing: { line: 360 }, alignment: AlignmentType.CENTER,
      children: [new TextRun({ text: "", font: FONT_HEI, size: 52 })] }),
    new Paragraph({ spacing: { line: 360 }, alignment: AlignmentType.CENTER,
      children: [new TextRun({ text: "定制班课程报告", font: FONT_HEI, size: 48, bold: true })] }),
    new Paragraph({ alignment: AlignmentType.CENTER,
      children: [new TextRun({ text: "", font: FONT_HEI, size: 44 })] }),
    new Paragraph({ alignment: AlignmentType.CENTER,
      children: [new TextRun({ text: "", font: FONT_HEI, size: 44 })] }),
    new Paragraph({ spacing: { line: 360 }, alignment: AlignmentType.CENTER,
      children: [new TextRun({ text: "", font: FONT_HEI, size: 32 })] }),
    new Table({
      width: { size: 6577, type: WidthType.DXA },
      columnWidths: [1819, 4758],
      rows: [
        mkRow("课    程：", "<<课程名称>>"),
        mkRow("学    院：", "<<学院>>"),
        mkRow("专    业：", "<<专业>>"),
        mkRow("班    级：", "<<班级>>"),
        mkRow("学生姓名：", "<<姓名>>"),
        mkRow("学    号：", "<<学号>>"),
        mkRow("指导教师：", "<<指导教师>>"),
        mkRow("提交日期：", "<<提交日期>>"),
      ],
    }),
    new Paragraph({ spacing: { before: 400 }, alignment: AlignmentType.CENTER,
      children: [new TextRun({ text: "<<提交日期>>", font: FONT_BODY, size: 24 })] }),
  ];
}

// ============================================================
// ABSTRACT PAGE
// Title: "摘  要" 黑体小二(size 36) bold, centered, 单倍行距
// Body: 小四号宋体, 首行缩进2字符, 1.5倍行距
// Keywords label: 黑体小四(size 24) bold
// Keywords: 宋体小四, 词间"，"分隔
// ============================================================
function makeAbstract(abstractText, keywords) {
  return [
    new Paragraph({
      spacing: { before: 360, after: 240 },
      alignment: AlignmentType.CENTER,
      children: [new TextRun({ text: "摘  要", font: FONT_HEI, size: 36, bold: true })],
    }),
    p(abstractText, { indent: true }),
    new Paragraph({ spacing: { before: 200 }, children: [new TextRun({ text: "" })] }), // empty line
    new Paragraph({
      spacing: { line: 360 },
      indent: { firstLine: 640 },
      children: [
        new TextRun({ text: "关键词", font: FONT_HEI, size: 24, bold: true }),
        new TextRun({ text: keywords, font: FONT_BODY, size: 24 }),
      ],
    }),
  ];
}

// ============================================================
// ENGLISH ABSTRACT PAGE (optional)
// ============================================================
function makeEnAbstract(abstractText, keywords) {
  return [
    new Paragraph({
      spacing: { before: 360, after: 240 },
      alignment: AlignmentType.CENTER,
      children: [new TextRun({ text: "Abstract", font: "Times New Roman", size: 36, bold: true })],
    }),
    new Paragraph({
      spacing: { line: 360 },
      indent: { firstLine: 640 },
      alignment: AlignmentType.JUSTIFIED,
      children: [new TextRun({ text: abstractText, font: "Times New Roman", size: 24 })],
    }),
    new Paragraph({ spacing: { before: 200 }, children: [new TextRun({ text: "" })] }),
    new Paragraph({
      spacing: { line: 360 },
      indent: { firstLine: 640 },
      children: [
        new TextRun({ text: "Key Words", font: FONT_HEI, size: 24, bold: true }),
        new TextRun({ text: keywords, font: "Times New Roman", size: 24 }),
      ],
    }),
  ];
}

// ============================================================
// APPENDIX — Glossary table (术语表)
// ============================================================
function makeGlossary() {
  const W1 = 2500, W2 = 2000, W3 = CONTENT_W - W1 - W2;
  const th = (t, w) => tc(t, w, { bold: true, shading: "D9E2F3" });

  return [
    h1("附录A　术语表"),
    tableCaption("表A.1 术语表"),
    new Table({
      width: { size: CONTENT_W, type: WidthType.DXA },
      columnWidths: [W1, W2, W3],
      rows: [
        new TableRow({ children: [th("术语", W1), th("英文", W2), th("释义", W3)] }),
        // Add 5+ data rows per the course topic
        new TableRow({ children: [tc("术语1", W1), tc("English1", W2), tc("释义1", W3)] }),
        new TableRow({ children: [tc("术语2", W1), tc("English2", W2), tc("释义2", W3)] }),
        new TableRow({ children: [tc("术语3", W1), tc("English3", W2), tc("释义3", W3)] }),
        new TableRow({ children: [tc("术语4", W1), tc("English4", W2), tc("释义4", W3)] }),
        new TableRow({ children: [tc("术语5", W1), tc("English5", W2), tc("释义5", W3)] }),
      ],
    }),
  ];
}

// ============================================================
// APPENDIX — Grading table (指导教师评阅意见表)
// ============================================================
function makeGradingTable() {
  const W1 = 2500, W2 = 3500, W3 = 1500, W4 = CONTENT_W - W1 - W2 - W3;
  const th = (t, w) => tc(t, w, { bold: true, shading: "D9E2F3" });

  return [
    h1("附录B　指导教师评阅意见表"),
    tableCaption("表B.1 指导教师评阅意见表"),
    new Table({
      width: { size: CONTENT_W, type: WidthType.DXA },
      columnWidths: [W1, W2, W3, W4],
      rows: [
        new TableRow({ children: [th("评阅项目", W1), th("评阅内容", W2), th("得分", W3), th("备注", W4)] }),
        new TableRow({ children: [tc("报告结构", W1), tc("章节完整、结构清晰", W2), tc("", W3), tc("", W4)] }),
        new TableRow({ children: [tc("内容质量", W1), tc("理论准确、内容充实", W2), tc("", W3), tc("", W4)] }),
        new TableRow({ children: [tc("格式规范", W1), tc("页面、字体、图表规范", W2), tc("", W3), tc("", W4)] }),
        new TableRow({ children: [tc("学习心得", W1), tc("体会真实、总结到位", W2), tc("", W3), tc("", W4)] }),
      ],
    }),
  ];
}

// ============================================================
// DOCUMENT ASSEMBLY
// 3 sections: Cover → Front Matter (Abstract+TOC) → Body (Ch1-5+Appendix)
// ============================================================
async function main() {
  const doc = new Document({
    styles: {
      default: {
        document: {
          run: { font: FONT_BODY, size: 24 },
        },
      },
      paragraphStyles: [
        {
          id: "Heading1", name: "Heading 1", basedOn: "Normal", next: "Normal", quickFormat: true,
          run: { size: 36, bold: true, font: FONT_HEI },
          paragraph: { spacing: { before: 240, after: 240 }, alignment: AlignmentType.CENTER, outlineLevel: 0 },
        },
        {
          id: "Heading2", name: "Heading 2", basedOn: "Normal", next: "Normal", quickFormat: true,
          run: { size: 30, bold: true, font: FONT_HEI },
          paragraph: { spacing: { before: 240, after: 240 }, outlineLevel: 1 },
        },
        {
          id: "Heading3", name: "Heading 3", basedOn: "Normal", next: "Normal", quickFormat: true,
          run: { size: 28, bold: true, font: FONT_HEI },
          paragraph: { spacing: { before: 240, after: 240 }, outlineLevel: 2 },
        },
      ],
    },
    sections: [
      // ====== Section 1: Cover page (no headers/footers) ======
      {
        properties: {
          page: {
            size: { width: PAGE_W, height: PAGE_H },
            margin: { top: MARGIN, right: MARGIN, bottom: MARGIN, left: MARGIN, header: H_MARGIN, footer: F_MARGIN },
          },
        },
        children: makeCover(),
      },

      // ====== Section 2: Front Matter — Abstract + TOC ======
      {
        properties: {
          page: {
            size: { width: PAGE_W, height: PAGE_H },
            margin: { top: MARGIN, right: MARGIN, bottom: MARGIN, left: MARGIN, header: H_MARGIN, footer: F_MARGIN },
          },
        },
        headers: {
          default: new Header({
            children: [new Paragraph({
              alignment: AlignmentType.CENTER,
              border: { bottom: { style: BorderStyle.SINGLE, size: 6, color: "auto", space: 1 } },
              children: [new TextRun({ text: "成都东软学院定制班课程报告", font: FONT_BODY, size: 21 })],
            })],
          }),
        },
        footers: {
          default: new Footer({
            children: [new Paragraph({
              alignment: AlignmentType.CENTER,
              children: [new TextRun({ children: [PageNumber.CURRENT], font: "宋体", size: 18 })],
            })],
          }),
        },
        children: [
          // Chinese Abstract
          ...makeAbstract("在此处填写中文摘要内容。", "关键词1，关键词2，关键词3"),
          new Paragraph({ children: [new PageBreak()] }),
          // English Abstract (optional)
          ...makeEnAbstract("Enter English abstract content here.", "keyword1, keyword2, keyword3"),
          new Paragraph({ children: [new PageBreak()] }),
          // Table of Contents
          new Paragraph({
            spacing: { before: 360, after: 360 },
            alignment: AlignmentType.CENTER,
            children: [new TextRun({ text: "目　录", font: FONT_HEI, size: 36, bold: true })],
          }),
          new TableOfContents("目录", { hyperlink: true, headingStyleRange: "1-3" }),
          new Paragraph({ children: [new PageBreak()] }),
        ],
      },

      // ====== Section 3: Body (Chapters 1-5 + Appendix) ======
      {
        properties: {
          page: {
            size: { width: PAGE_W, height: PAGE_H },
            margin: { top: MARGIN, right: MARGIN, bottom: MARGIN, left: MARGIN, header: H_MARGIN, footer: F_MARGIN },
          },
        },
        headers: {
          default: new Header({
            children: [new Paragraph({
              alignment: AlignmentType.CENTER,
              border: { bottom: { style: BorderStyle.SINGLE, size: 6, color: "auto", space: 1 } },
              children: [new TextRun({ text: "成都东软学院定制班课程报告", font: FONT_BODY, size: 21 })],
            })],
          }),
        },
        footers: {
          default: new Footer({
            children: [new Paragraph({
              alignment: AlignmentType.CENTER,
              children: [new TextRun({ children: [PageNumber.CURRENT], font: "Times New Roman", size: 18 })],
            })],
          }),
        },
        children: [
          // Chapter content functions — define these per report
          ...makeCh1(),
          ...makeCh2(),
          ...makeCh3(),
          ...makeCh4(),
          ...makeCh5(),
          // Appendix
          ...makeGlossary(),
          ...makeGradingTable(),
        ],
      },
    ],
  });

  const buffer = await Packer.toBuffer(doc);
  fs.writeFileSync("<<输出文件名>>.docx", buffer);
  console.log("Document generated successfully!");
  console.log(`File size: ${(buffer.length / 1024).toFixed(1)} KB`);
}

main().catch(err => { console.error(err); process.exit(1); });
```

## Quick Templates for Content Writing

### Body paragraph with Chinese quotes:
```javascript
p(`C++作为编译型语言，兼具C语言的高效性与面向对象编程的灵活性，支持"多范式"编程。`, { indent: true })
```

### Body paragraph with bold lead-in:
```javascript
new Paragraph({
  spacing: { line: 360 },
  indent: { firstLine: 640 },
  alignment: AlignmentType.JUSTIFIED,
  children: [
    new TextRun({ text: "类与对象", font: FONT_BODY, size: 24, bold: true }),
    new TextRun({ text: "是面向对象编程的基础。课程首先介绍...", font: FONT_BODY, size: 24 }),
  ],
})
```

### Data table with header:
```javascript
const W1 = 3000, W2 = 3000, W3 = CONTENT_W - W1 - W2;
tableCaption("表1.1 C++与其他语言对比"),
new Table({
  width: { size: CONTENT_W, type: WidthType.DXA },
  columnWidths: [W1, W2, W3],
  rows: [
    new TableRow({ children: [
      tc("特性", W1, { bold: true, shading: "D9E2F3" }),
      tc("C++", W2, { bold: true, shading: "D9E2F3" }),
      tc("Java", W3, { bold: true, shading: "D9E2F3" }),
    ]}),
    new TableRow({ children: [
      tc("内存管理", W1),
      tc("手动(delete/free)", W2),
      tc("自动(GC)", W3),
    ]}),
  ],
}),
```

### Code block:
```javascript
new Paragraph({
  spacing: { before: 120, line: 300 },
  children: [new TextRun({ text: "// C++ code example", font: "Times New Roman", size: 21 })],
}),
```

### Chapter structure:
```javascript
function makeCh1() {
  return [
    h1(`第1章　绪论`),
    h2("1.1 课程概述"),
    h3("1.1.1 学科背景与发展"),
    p(`正文内容...`, { indent: true }),
  ];
}
```
