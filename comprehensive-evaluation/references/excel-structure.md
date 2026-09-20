# Excel Output Structure

The generated workbook has one worksheet with a fixed two-row header (rows 1-2) followed by
student rows starting at row 3. The table spans columns A-O.

## Rows 1-2

Row 1:

- A1: 学号 (merged A1:A2)
- B1: 姓名 (merged B1:B2)
- C1:E1: 思想品德表现（C1）
- F1: 课程学习成绩（C2）
- G1:I1: 发展素质（C3）
- J1:K1: 综合测评成绩(C)C=C1×10%+C2×70%+C3×20%
- L1: 体侧成绩 (merged L1:L2)
- M1: 课程成绩排名（C2） (merged M1:M2)
- N1: 奖学金类型 (merged N1:N2)
- O1: 备注 (merged O1:O2)

Row 2:

- C2: 基准分
- D2: 加减分
- E2: 小计
- F2: 学分加权平均分
- G2: 基准分
- H2: 加减分
- I2: 小计
- J2: 合计
- K2: 名次

Merged ranges:

```text
A1:A2
B1:B2
C1:E1
G1:I1
J1:K1
L1:L2
M1:M2
N1:N2
O1:O2
```

Do not merge F1:F2: F1 contains `课程学习成绩（C2）` and F2 contains `学分加权平均分`.

## Header Style

- Header cells: 宋体, 12 pt, bold, centered horizontally and vertically, wrap text.
- Thin borders around the two header rows.
- Data rows begin at row 3. For student n, the row is n+2.
- For every student data row, set C (C1 基准分) to `80` and G (C3 基准分) to `65`.
- Write D from the C1 personal bonus and deduction columns, without C1 rule-table text or
  the C1 bottom total text.
- Write E from the C1 bottom `合计`, capped at 100. If that total is blank, calculate
  `80 + C1加分 - C1减分`, capped at 100.
- For the C3 `主动学习加分` row, write one aggregated item such as `自主学习+2` or
  `自主学习+1.5` in H instead of copying the individual activity entries.
- For the C3 `体质测试加分` row, write exactly `体测良好+2` or `体测优秀+3` in H
  instead of copying the student's explanation.
- Use generous height for rows with long D or H content and enable wrap text on D and H.

Recommended column widths:

```text
A 14.07  B 9.6  C 13  D 37.2  E 9.6  F 21.53
G 9.6    H 43.07  I 9.6  J 13  K 13
L 13     M 13  N 18.27  O 22.8
```
