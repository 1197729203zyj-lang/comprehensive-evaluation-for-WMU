# Word Structure and Parsing Notes

The same Word template is used for every student. Read this reference when a document does not parse as expected or when the first attempt mixes C1, homepage, or standard-table values into H.

## Document Layout

Typical top-level tables and paragraphs in the main `.doc/.docx`:

1. Front matter paragraphs: 姓名, 学号, 班级, 年度.
2. Homepage scoring card `温州医科大学本专科学生素质综合测评评分表` containing C1/C2/C3 summary columns such as 基准分, 加减分/加分, 小计, 合计.
3. `分表一 思想品德表现测评（C1）` plus its detail table.
4. Paragraph `分表二 发展素质测评（C3）`, followed by the C3 detail table.
5. `支撑材料粘贴处` table.

Locate the C3 detail table by finding the paragraph containing `发展素质测评（C3）` and selecting the next table whose text includes both `个人加分项目及分值` and `合计`.

## C3 Detail Table

- Header row: `加分依据 | 个人加分项目及分值`.
- The first column contains long rules/criteria and nested point-scale tables. Do not copy its values.
- The final personal column `个人加分项目及分值` is where students write their own content.
- The table can repeat `加分依据 | 个人加分项目及分值` header rows across page breaks; skip those header-only rows.
- Because cells are merged or repeated across physical rows, `python-docx` may return the same content many times. Deduplicate identical non-empty cell values before building H.
- Bottom row: `合计 | 65+...=...`, which is the only acceptable Word subtotal source for I.

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

- Copying homepage C3 `加分`/`小计` into H/I.
- Copying C1 detail content because its table has a similar `个人加分项目及分值` header.
- Treating repeated merged-cell content as multiple bonuses.
- Treating standard table scores such as `10`, `6`, `4`, `2` in the `加分依据` columns as the student's H content.
- Splitting an award name at an artificial line break and then making the trailing fragment a separate H item.
