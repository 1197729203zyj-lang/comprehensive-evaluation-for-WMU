---
name: comprehensive-evaluation-c3
description: Fill a student comprehensive-evaluation Excel workbook from each student's Word "综合测评记载册" by extracting only the 分表二 C3 bonuses into column H and its derived C3 subtotal into column I. Use when asked to process a batch of student Word/zip/rar evaluation files into an Excel score table.
---

# Comprehensive Evaluation C3

Process a batch of student comprehensive-evaluation materials into one Excel workbook.

## Goal

For each student, fill one Excel data row with:

- A: 学号
- B: 姓名
- H: C3 bonus content from the Word's `分表二 发展素质测评（C3）` table
- I: C3 subtotal from the same 分表二, never from the homepage score card

## Input Handling

- The batch root usually mixes `.doc`, `.docx`, `.zip`, `.rar`, and already-extracted folders. Treat each top-level item as one student.
- Find the student's actual `附件1`/`记载册` Word file, not `支撑材料` or support attachments. Filenames may be garbled in old archives; choose the file whose content contains the C3 table and whose name/ID matches the student.
- Convert old binary `.doc` files to `.docx` with the local Word converter (Wordconv) or another available converter before parsing. This often requires an approval because Word opens in the real desktop session.
- Parse `.docx` with `python-docx` or equivalent. Read supporting layout details from [references/word-structure.md](references/word-structure.md) before parsing unfamiliar files.

## Extraction Rules

- Ignore the homepage `温州医科大学本专科学生素质综合测评评分表` entirely for C3 values.
- Ignore C1 and all C1 detail content.
- Extract only from the later `分表二 发展素质测评（C3）` table.
- Within that table, read only the column headed `个人加分项目及分值`.
- Remove empty cells and no-bonus markers such as `0` and `无`.
- Identical content repeated by merged cells must be deduplicated, not counted multiple times.
- Put each bonus item in H. Separate bonus items with the full-width semicolon `；`. Keep commas that are part of a description.
- If a cell contains only a score and no description, such as `2` or `+2`, write the score as `+2`-style text and apply red font to that score segment only.
- Do not read numeric values from the `加分依据`/standard tables as bonuses.

## Subtotal

- Use the 分表二 bottom `合计` row, formatted like `65+...=...`, as I.
- Cap I at 100 because the evaluation rule is `C3≤100`.
- If the bottom total is blank, compute `65 + points extracted from the 分表二 H items`, then cap at 100.
- Never substitute the homepage score card's C3 `加分` or `小计` values.

## Excel Output

- Write each student on a new row.
- If a student ID already exists in the target workbook, do not write that student twice.
- Preserve the workbook's existing layout/styles; set H cells to wrap text and give rows enough height for long H content.
- Keep red coloring limited to point-only bonus segments rather than formatting the whole cell red.

## Verification

- Confirm the number of filled student rows equals the number of top-level student items.
- Confirm student IDs are unique and the set of IDs matches the batch folder exactly.
- Spot-check A, B, H, and I against the original Word files.
- If a Word contains only homepage totals and no 分表二 entries, leave H empty and report that student separately rather than inventing content.

See [references/word-structure.md](references/word-structure.md) for the Word table map and common parsing pitfalls.
