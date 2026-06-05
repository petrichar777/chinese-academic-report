---
name: chinese-academic-report
description: Generate Chinese academic/end-of-semester reports as .docx files following 成都东软学院定制班课程报告撰写规范. Use this skill whenever the user mentions 期末报告, 课程报告, 学术总结, 定制班报告, docx报告, 中文Word报告, 期末总结报告, or needs to create a formatted Chinese academic document with cover page, abstract, table of contents, chapters, and appendix. Also trigger on any request involving generating structured Chinese academic reports in Word format, even without explicit keywords.
---

# Chinese Academic Report Generator

Generate formatted Chinese academic reports (.docx) strictly following the **成都东软学院定制班课程报告撰写规范** (Chengdu Neusoft University Custom Class Course Report Writing Specification). This specification serves as the **default** — only deviate if the user explicitly provides alternative formatting requirements.

## Workflow

### Phase 1: Requirements Gathering

1. **Ask the user for:**
   - Course name (课程名称) — used in cover, header, and report title
   - Student info — 学院, 专业, 班级, 姓名, 学号
   - Advisor name (指导教师)
   - Submission date (提交日期) — e.g., "2025年6月"
   - Report language: Chinese only, or Chinese + English abstract
   - Any custom topic focus or content preference

2. **Design the report outline** following the spec's required structure:
   - **前置部分 (Front Matter):** Cover → Chinese Abstract → English Abstract (optional) → Table of Contents
   - **主体部分 (Main Body):** 5 chapters:
     - Chapter 1: 绪论 (Introduction — purpose, scope, methodology, background)
     - Chapter 2: Core Technology/Knowledge Deep Dive
     - Chapter 3: Advanced Topics or Application Scenarios
     - Chapter 4: Frontier Trends or Case Studies
     - Chapter 5: 结论 (Conclusion + 学习心得)
   - **附录 (Appendix):** 术语表 (Glossary) + 指导教师评阅意见表 (Grading Table)

3. **Prioritize theory and analysis** — avoid requiring actual coding projects. Use comparison tables and diagrams (described in text) where possible.

4. **Wait for user approval** before generating code.

### Phase 2: Write Generation Script

Use Node.js with the `docx` npm package. Copy all helper functions from `references/helper-templates.md`.

**Read these reference files first:**
- `references/format-specs.md` — Complete specification (every detail from the 撰写规范)
- `references/page-setup.md` — Page size, margins, header/footer distances
- `references/common-pitfalls.md` — Error patterns and fixes

**Format specification summary (成都东软学院定制班课程报告撰写规范):**

| Element | Font (中文) | Font (数字/英文) | Size | Weight | Other |
|---------|------------|-----------------|------|--------|-------|
| Body text (正文) | 宋体 | Times New Roman | 小四 12pt (size 24) | Normal | 1.5倍行距(line 360), 首行缩进2字符(640) |
| H1 (一级/章) | 黑体 | Times New Roman | 小二 18pt (size 36) | Bold | 居中, 段前1行段后1行, 单倍行距, 另起页 |
| H2 (二级/节) | 黑体 | Times New Roman | 小三 15pt (size 30) | Bold | 居左, 段前1行段后1行, 单倍行距 |
| H3 (三级/条) | 黑体 | Times New Roman | 四号 14pt (size 28) | Bold | 居左, 段前1行段后1行, 单倍行距 |
| H4 (四级/项) | 宋体 | Times New Roman | 小四 12pt (size 24) | Normal | 首行缩进2字符, 段前1行段后1行, 1.5倍行距 |
| Table cell | 宋体 | — | 五号 10.5pt (size 21) | Normal | Center aligned |
| Table header | 宋体 | — | 五号 10.5pt (size 21) | Bold | Shading: D9E2F3, CLEAR |
| Table caption (表题) | 宋体 | — | 五号 10.5pt (size 21) | Normal | ABOVE table, centered |
| Figure caption (图题) | 宋体 | — | 五号 10.5pt (size 21) | Normal | BELOW figure, centered |
| Code block | 宋体 | Times New Roman | 五号 10.5pt (size 21) | Normal | 多倍行距1.25, line 300 |
| Header (页眉) | 宋体 | — | 五号 10.5pt (size 21) | Normal | "成都东软学院定制班课程报告", centered |
| Footer (页脚/页码) | — | Times New Roman | 小五 9pt (size 18) | Normal | Centered |

**Page setup:** A4 (11906 x 16838 DXA), margins **25mm (984 DXA)**, header 1.5cm (567), footer 1.75cm (661).

**Document assembly (Section blocks):**
1. Cover page (no headers/footers)
2. Chinese Abstract (with headers/footers, Roman page numbers)
3. English Abstract (optional, same as above)
4. Table of Contents (with headers/footers, Roman page numbers continued)
5. Body: Chapters 1-5 (with headers/footers, Arabic page numbers restarted from 1)
6. Appendix (continuation of body section)

**Cover page structure:**
- School name "成都东软学院" (黑体 size 52 bold, centered)
- "定制班课程报告" (黑体 size 48 bold, centered)
- Course info table: 8 rows — 课程名称/学院/专业/班级/姓名/学号/指导教师/提交日期
  - Label column: 1819 DXA, 黑体 三号 size 32, centered
  - Value column: 4758 DXA, 宋体 三号 size 32, centered, bottom border only
- Total table width: 6577 DXA, centered

**Headers & Footers (自摘要页起):**
- Header: "成都东软学院定制班课程报告" (宋体 五号 size 21, centered, bottom border single line)
- Footer (前置部分): Roman numerals (宋体 小五 size 18, centered)
- Footer (正文): Arabic numerals (Times New Roman 小五 size 18, centered, restart from 1)

**Abstract page (摘要):**
- Title: "摘  要" (黑体 小二 size 36 bold, centered, 2 spaces between characters)
- Body: 小四号宋体, 首行缩进2字符, 1.5倍行距
- Keywords label: "关键词" (黑体 小四 size 24 bold), followed by 宋体 小四 keywords separated by "，"

**TOC (目录):**
- Title: "目  录" (黑体 小二 size 36 bold, centered, 2 spaces)
- Level 1 entries: 小四号黑体 (size 24), 1.5倍行距
- Level 2/3 entries: 小四号宋体 (size 24), 1.5倍行距
- Level 3: left indent 2 characters
- Page numbers: 小四号 Times New Roman (size 24), dot leaders

**Chinese quote handling (CRITICAL):**
- For body text `p()` calls: ALWAYS use backtick template literals to avoid Chinese quotes breaking JS strings
- For `tc()` calls: regular string quotes work fine

### Phase 3: Run & Debug

1. Install dependencies: `npm install docx` (if `node_modules/` doesn't exist)
2. Run: `node generate_xxx.js`
3. Common errors: see `references/common-pitfalls.md`
4. Verify: file size 50-70KB for ~35-page report, starts with PK (ZIP magic bytes)

## Reference Files

- `references/format-specs.md` — **Complete format specification** — every detail from 成都东软学院定制班课程报告撰写规范 (read this FIRST)
- `references/helper-templates.md` — Complete helper function code with all constants, functions, and assembly template
- `references/page-setup.md` — Page size, margin, and header/footer distance reference
- `references/common-pitfalls.md` — Error patterns, fixes, and verification checklist
