# Word Structure and Parsing Notes

The same Word template is used for every student. Read this reference when a document does not parse as expected or when the extraction mixes C1 values into H, homepage values into E/I, or rule-table values into D/H.

## Document Layout

Typical top-level tables and paragraphs in the main `.doc/.docx`:

1. Front matter paragraphs: 姓名, 学号, 班级, 年度.
2. Homepage scoring card `温州医科大学本专科学生素质综合测评评分表` containing C1/C2/C3 summary columns such as 基准分, 加减分/加分, 小计, 合计.
3. `分表一 思想品德表现测评（C1）` plus its detail table, including separate personal bonus and deduction columns.
4. Paragraph `分表二 发展素质测评（C3）`, followed by the C3 detail table.
5. `支撑材料粘贴处` table.

Locate the C3 detail table by finding the paragraph containing `发展素质测评（C3）` and selecting the next table whose text includes both `个人加分项目及分值` and `合计`.

## C1 Detail Table

- Header shape: `加减分项目 | 加分依据 | 个人加分项目及分值 | 减分依据 | 个人减分项目及分值`.
- Read only `个人加分项目及分值` and `个人减分项目及分值`.
- The adjacent rule columns contain the scoring standards and must never be copied into Excel D.
- Skip the `分值小计` row and the bottom `合计` row when building D.
- Deduplicate identical non-empty values returned repeatedly because of merged cells.
- Multiple items in one personal cell are commonly separated by line breaks. Join independent
  items with `；`, but merge a line that is only a wrapped continuation of the same item.
- A value in `个人减分项目及分值` is always a deduction. Preserve `-` or `减`; add a negative
  sign when the student provided only an unsigned score. Never emit a deduction as `+2`.
- The C1 bottom row is `合计 | 80+...-...=...`. Its result is the preferred source for E,
  subject to `C1≤100`; it must not be copied into D.
- If the C1 bottom total is blank, compute E as `80 + C1 bonus points - absolute C1 deduction
  points`, capped at 100.
- Never use the homepage score card's C1 `加分` or `小计` for E.

## C3 Detail Table

- Header row: `加分依据 | 个人加分项目及分值`.
- The first column contains long rules/criteria and nested point-scale tables. Do not copy its values.
- The final personal column `个人加分项目及分值` is where students write their own content.
- The table can repeat `加分依据 | 个人加分项目及分值` header rows across page breaks; skip those header-only rows.
- Because cells are merged or repeated across physical rows, `python-docx` may return the same content many times. Deduplicate identical non-empty cell values before building H.
- Bottom row: `合计 | 65+...=...`, which is the only acceptable Word subtotal source for I.

## C3 Active Learning Row

- Locate the row whose `加分依据` contains `主动学习加分`, commonly `5．主动学习加分。主动积极参与各种大型讲座和活动...`.
- This row is an exception to the normal itemized H output. Do not copy its lecture or activity list.
- Read its `个人加分项目及分值` cell, parse each point value, sum the entries, and cap the row at 2.
- Count only signed or explicitly scored expressions such as `+0.5`, `-0.5`, `加0.5`, or `0.5分`. Do not parse list numbering (`1.`, `2.`), dates, item counts, or unrelated numbers in activity names.
- Replace the whole row with one H item labeled `自主学习+分值`. For example, seven entries of `+0.5` each must become `自主学习+2`, not a seven-item list.
- Keep the normal number format without trailing zeros: `+2`, `+1.5`, `+1`, `+0.5`.
- If the row is empty, zero, or has no positive contribution after aggregation, emit nothing for that row.
- Keep the 2-point cap local to this row; do not use it as the overall C3 cap.

## C3 Physical Fitness Row

- Locate the row whose `加分依据` contains `体质测试加分`, commonly `4．体质测试加分。大学生国家体质健康标准测试获良好等级者加2分，优秀等级者加3分。`.
- This row is also an exception to normal itemized H output. Do not copy the student's explanation.
- Normalize the row to exactly one item:
  - `优秀` or `+3` means `体测优秀+3`.
  - `良好` or `+2` means `体测良好+2`.
- Accept full-width and half-width plus signs and surrounding spaces. If the cell contains conflicting values, prefer the higher `优秀`/`+3` result.
- If the cell contains neither a valid level nor a valid score, emit nothing for that row.
- Keep the normalized value local to this row and count it only once when calculating a fallback I subtotal.

## Cell Content Examples

Descriptive bonus:

```text
体测良好+2
担任班级学习委员+3
```

Point-only bonus (no description, red in Excel):

```text
2
+2
4
```

No bonus markers to skip:

```text
0
无
```

Multiple bonuses inside one cell are often separated by line breaks:

```text
医创赛院级参赛奖+0.5
提案大赛参与奖+0.5
```

Long award names can also wrap across lines. Merge a continuation line with its award rather than treating it as a separate bonus.

## Common Mistakes

- Copying C1 rule-table scores from `加分依据` or `减分依据` into D.
- Missing the `个人减分项目及分值` column or writing its deductions as positive numbers.
- Mixing the C1 and C3 personal columns.
- Copying the homepage C1 `加分` or `小计` into E instead of using the 分表一 bottom `合计`.
- Copying the C1 bottom `合计` text into D instead of using it only to calculate E.
- Copying homepage C3 `加分`/`小计` into H/I.
- Using C1 personal entries as H, or C3 personal entries as D, because both tables have a `个人加分项目及分值` column.
- Treating the C1 `分值小计` or `合计` row as a personal bonus item in D.
- Treating repeated merged-cell content as multiple bonuses.
- Copying the full lecture/activity list from the `主动学习加分` row into H instead of writing one aggregated `自主学习+分值` item.
- Forgetting to cap the `主动学习加分` row at 2 or applying that 2-point cap to all C3 items.
- Using the rule title `主动学习` in H instead of the required output label `自主学习`.
- Copying the raw explanation from the `体质测试加分` row instead of writing `体测良好+2` or `体测优秀+3`.
- Mapping `优秀` to `+2`, mapping `良好` to `+3`, or counting a normalized physical-fitness result more than once.
- Treating standard table scores such as `10`, `6`, `4`, `2` in the `加分依据` columns as the student's H content.
- Splitting an award name at an artificial line break and then making the trailing fragment a separate H item.
