---
name: 温医大综测
description: Create or keep the standard two-row A-O header of a 综合测评 Excel workbook, then fill each student from their Word 综合测评记载册 starting at row 3 with A=学号, B=姓名, D=分表一 C1 personal bonus/deduction items, E=C1 subtotal, H=分表二 C3 bonus content, and I=C3 subtotal. Use when asked to process student Word/zip/rar evaluation files into an Excel score table.
---

# Comprehensive Evaluation C1/C3

Process a batch of student comprehensive-evaluation materials into one Excel workbook.

## Goal

For each student, fill one Excel data row with:

- A: 学号
- B: 姓名
- C: C1 基准分 80
- D: C1 personal bonus and deduction items from `分表一 思想品德表现测评（C1）`
- E: C1 subtotal from the same 分表一, capped at 100
- F: 学分加权平均分 from the supplied academic-ranking workbook
- G: C3 基准分 65
- H: C3 bonus content from the Word's `分表二 发展素质测评（C3）` table
- I: C3 subtotal from the same 分表二, never from the homepage score card
- J: 综合测评成绩, calculated from C1, C2, and C3 after all source data is filled
- K: 综合测评成绩 ranking, calculated from J after all student rows are complete
- L: 体测成绩 from the `学期成绩` column in the supplied physical-fitness workbook
- M: 学分加权平均分排名 from the supplied academic-ranking workbook

The output workbook must contain the standard two-row header template in rows 1-2.
Student data starts at row 3: write the first student to row 3, the second to row 4,
and continue downward in the corresponding columns.

## Input Handling

- The batch root usually mixes `.doc`, `.docx`, `.zip`, `.rar`, and already-extracted folders. Treat each top-level item as one student.
- When an academic-ranking `.xlsx` workbook is supplied, use its `总表` worksheet as the
  authoritative source for C2 values. Match each student by exact `学号` first; use `姓名`
  only when it identifies exactly one record. From the columns headed `学分加权平均分` and
  `学分加权平均分排名`, write the values to F and M respectively. Keep both values numeric.
  Do not use similarly named fields such as `算术平均分` or `平均绩点`. If no unique source
  record is found, leave F and M blank and report the student rather than guessing.
- When a physical-fitness `.xlsx` workbook is supplied, locate the worksheet with the
  headers `学号`, `姓名`, and `学期成绩`. Match each student by exact `学号` first; use
  `姓名` only when it identifies exactly one record. Copy `学期成绩` to L (体测成绩).
  Preserve numeric grades as numbers and preserve nonnumeric statuses such as `保健班` as
  their original text. Do not calculate L from first- or second-semester values. If no unique
  source record is found, leave L blank and report the student rather than guessing.
- Never align students by row number, source-file order, or name alone when an ID is
  available. For every matched record, compare the source `姓名` with the Word-derived
  student name after trimming whitespace. If the same ID has conflicting names across
  sources, leave the affected imported fields blank and report the conflict for review.
- Find the student's actual `附件1`/`记载册` Word file, not `支撑材料` or support attachments. Filenames may be garbled in old archives; choose the file whose content contains the C3 table and whose name/ID matches the student.
- Convert old binary `.doc` files to `.docx` with the local Word converter (Wordconv) or another available converter before parsing. This often requires an approval because Word opens in the real desktop session.
- Parse `.docx` with `python-docx` or equivalent. Read supporting layout details from [references/word-structure.md](references/word-structure.md) before parsing unfamiliar files.
- If the target workbook is read-only, use it only as a reference. Do not change file attributes, replace it in place, or attempt to bypass the restriction. Create a separate writable copy only when the user requests output.

## C1 Extraction (D)

- Locate `分表一 思想品德表现测评（C1）` and select its following detail table. The table header includes `加减分项目`, `加分依据`, `个人加分项目及分值`, `减分依据`, and `个人减分项目及分值`.
- Read only the two personal columns: `个人加分项目及分值` and `个人减分项目及分值`.
- Ignore the rule columns `加分依据` and `减分依据`, the `分值小计` row, the bottom `合计` row, and no-content markers such as empty cells, `0`, `无`, `/`, and `-` when building D.
- Deduplicate identical content repeated by merged cells.
- Preserve complete descriptions. If one cell contains multiple independent bonus or deduction items, join them with the full-width semicolon `；`. Keep commas that belong to an item; merge an artificial line wrap into the same item.
- Write C1 bonus items first, then C1 deduction items. If there are no C1 items, leave D empty.
- Every C1 deduction must be represented as negative. Keep an existing `-` or `减` sign. If a deduction cell contains only an unsigned number such as `2` or `2分`, write `-2` or `-2分`. If it combines a description with an unsigned score, insert `-` before the score without altering the description.
- For a C1 bonus cell containing only an unsigned number such as `2` or `2分`, write a leading `+` (`+2` or `+2分`).
- Apply red font only to a score-only segment such as `+2` or `-2`, not to the entire D cell.

## C1 Subtotal (E)

- Use the 分表一 bottom `合计` row, formatted like `80+...-...=...`, as E.
- Cap E at 100 because the evaluation rule is `C1≤100`.
- If the bottom total is blank, compute `80 + extracted C1 bonus points - absolute extracted C1 deduction points`, then cap at 100.
- Never substitute the homepage score card's C1 `加分` or `小计` values.
- If a Word contains no usable 分表一 detail table, leave D and E empty and report that student separately rather than inventing content.

## C3 Extraction (H, I)

- Ignore the homepage `温州医科大学本专科学生素质综合测评评分表` entirely for C3 values.
- Ignore C1 and all C1 detail content.
- Extract only from the later `分表二 发展素质测评（C3）` table.
- Within that table, read only the column headed `个人加分项目及分值`.
- Remove empty cells and no-bonus markers such as `0` and `无`.
- Identical content repeated by merged cells must be deduplicated, not counted multiple times.
- Put each bonus item in H. Separate bonus items with the full-width semicolon `；`. Keep commas that are part of a description.
- If a cell contains only a score and no description, such as `2` or `+2`, write the score as `+2`-style text and apply red font to that score segment only.
- Do not read numeric values from the `加分依据`/standard tables as bonuses.

## C3 Active Learning Aggregation

- In 分表二, identify the row whose `加分依据` text contains `主动学习加分`, usually row `5．主动学习加分`.
- Do not copy the individual lecture or activity entries from that row's `个人加分项目及分值` column into H.
- Parse all score entries in that cell, sum their effective values, and cap the row total at 2. Apply any explicit positive or negative sign before summing.
- Count only explicit score expressions such as `+0.5`, `-0.5`, `加0.5`, or `0.5分`; never sum list numbers such as `1.`, dates, attendance counts, or numbers embedded in an activity name.
- Emit exactly one replacement item in H, labeled `自主学习+...`, for example `自主学习+2` or `自主学习+1.5`.
- Format the aggregate without trailing zeros: `自主学习+2`, `自主学习+1.5`, `自主学习+1`, `自主学习+0.5`.
- If this row has no valid positive contribution after aggregation, omit the replacement item.
- This 2-point cap applies only to the `主动学习加分` row. Other C3 rows remain individually itemized in H.
- When the C3 bottom total is blank, use the capped `自主学习` value when calculating the fallback I subtotal.

## C3 Physical Fitness Normalization

- In 分表二, identify the row whose `加分依据` text contains `体质测试加分`, usually row `4．体质测试加分`.
- Do not copy the student's explanatory text from that row's `个人加分项目及分值` column into H.
- If the row contains `优秀` or a score of `+3` or `3`, emit exactly one H item labeled `体测优秀+3`.
- If the row contains `良好` or a score of `+2` or `2`, emit exactly one H item labeled `体测良好+2`.
- Match full-width or half-width plus signs and surrounding spaces. Use `优秀` or `+3` as the higher result when the cell contains conflicting values.
- If neither a valid level nor a valid score is present, omit the replacement item.
- Count the normalized `体测良好+2` or `体测优秀+3` value only once in the fallback I subtotal.

## C3 Subtotal (I)

- Use the 分表二 bottom `合计` row, formatted like `65+...=...`, as I.
- Cap I at 100 because the evaluation rule is `C3≤100`.
- If the bottom total is blank, compute `65 + effective C3 points after row-specific caps`, then cap at 100. Count the aggregated `自主学习` row only once.
- Never substitute the homepage score card's C3 `加分` or `小计` values.

## Excel Output

- Use a single worksheet. If the user supplies or the previous workflow already created a
  workbook whose rows 1-2 are the standard header, keep that layout. Otherwise create the
  full standard table from [references/excel-structure.md](references/excel-structure.md).
- Unless the user explicitly requests another filename, name the generated workbook `综测.xlsx`.
- Rows 1-2 are the fixed header for columns A-O. Create the exact labels and merges from
  the reference before writing data.
- Student data rows begin at row 3 and continue downward without gaps: student 1 is row 3,
  student 2 is row 4, and so on. Never put a student in row 1 or 2.
- Within each student row, write A=学号, B=姓名, C=80, D=C1加减分项目,
  E=C1小计, F=学分加权平均分, G=65, H=C3加分内容, I=C3小计, and
  L=体测成绩, M=学分加权平均分排名 in the columns that match the row 2 subheaders `学号`,
  `姓名`, C1 `基准分`, C1 `加减分`, C1 `小计`, `学分加权平均分`, C3 `基准分`,
  C3 `加减分`, C3 `小计`, `体侧成绩`, and `课程成绩排名（C2）`.
- Fill J and K last, only after E, F, and I have been populated for the entire batch.
  For each data row `r`, write the Excel formula
  `=IF(COUNT(Er,Fr,Ir)=3,Er*10%+Fr*70%+Ir*20%,"")` in J. This requires all
  three numeric components: C1=E, C2=F, and C3=I. Never calculate a partial
  composite score.
- Once J formulas are in place, write the Excel formula
  `=IF(Jr="","",RANK.EQ(Jr,$J$3:$J$lastDataRow,0))` in K for each data row.
  Rank highest J score as 1. Use the actual last student data row in place of
  `lastDataRow`; do not include header rows or unused rows in the ranking range.
- If a student ID already exists in the target workbook, do not write that student twice.
- When a writable workbook is being updated, set D and E from the C1 extraction rather than
  preserving stale or manually entered C1 project text or subtotal for that student.
- Preserve or reproduce the standard workbook layout/styles; set D and H cells to wrap text
  and give data rows enough height for long content.
- Do not apply any background fill color to the worksheet, including the two header rows and
  student data rows. Keep the workbook's background unfilled; use borders, font, alignment,
  merges, and wrapping as needed.
- Do not freeze panes or split the worksheet in a way that repeats rows 1-2 visually. The
  standard two-row header must appear only once when the workbook is opened.
- Keep red coloring limited to point-only score segments rather than formatting the whole cell red.

## Verification

- Confirm rows 1-2 match the standard header reference exactly.
- Confirm filled student rows begin at row 3 and that the number of filled data rows equals
  the number of top-level student items.
- Confirm student IDs are unique and the set of IDs matches the batch folder exactly.
- Spot-check A, B, D, E, H, and I against the original Word files.
- Confirm D contains only the two C1 personal columns and that deductions have a single
  negative sign.
- Confirm E matches the C1 bottom total or the fallback `80 + C1加分 - C1减分` calculation
  and is capped at 100.
- If a Word contains only homepage totals and no 分表二 entries, leave H empty and report that student separately rather than inventing content.

## Mandatory Final Self-Audit

Perform this audit last, after all source values and the J/K formulas have been written,
before delivering the workbook.

- Reconcile every populated student row, not just a sample: A (学号) and B (姓名) must match
  the student's Word record. Each imported F/M pair must trace to the same student ID in the
  academic-ranking workbook, and each L value must trace to the same student ID in the
  physical-fitness workbook. Confirm the matched source name agrees after whitespace cleanup.
- Check that target student IDs are unique and that no source value has been copied to a row
  with another student's ID or name. Treat unmatched IDs, duplicate IDs, and name conflicts as
  exceptions; leave the affected automated cells blank and report them separately.
- Review every J formula: it must calculate only when E, F, and I are all numeric; otherwise
  J must be blank. Review every K formula against the actual J data range and confirm it ranks
  J descending without headers or unused rows.
- Confirm there are no formula errors and that N (奖学金类型) and O (备注) remain available
  for manual review. Do not deliver until all exceptions are listed clearly.

See [references/word-structure.md](references/word-structure.md) for the Word table map and
common parsing pitfalls, and [references/excel-structure.md](references/excel-structure.md)
for the Excel rows 1-2 layout.
