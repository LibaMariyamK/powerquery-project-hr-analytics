# Power Query Capstone Project — BrightWave Solutions HR Analytics

## Scenario
You are a data analyst at **BrightWave Solutions**. HR has handed you a folder of
exports pulled from three different systems (biometric attendance machine, payroll
software, and the HR master records). Nothing has been cleaned. Your job is to turn
this mess into two management-ready reports:

1. An **Attendance Summary** (present days per employee per month, by department)
2. A **Payroll Budget vs Actual** report (department-wise, month-wise)

You must use Power Query for every step below — no manual Excel edits to the raw
files. Work through the tasks **in order**: several steps will not work (or will
give wrong results) until the previous cleaning step is done, exactly like a real
project.

## Files provided

| File | What it is | Condition |
|---|---|---|
| `RawData/Attendance_Jan.csv` | January attendance log | Messy |
| `RawData/Attendance_Feb.csv` | February attendance log | Messy |
| `RawData/Attendance_Mar.csv` | March attendance log | Messy |
| `RawData/Payroll_Raw.csv` | Jan–Mar payroll export, all months in one file | Messy |
| `Employee_Master.xlsx` | Employee reference table (ID, name, dept, designation, branch, salary) | Clean — use as your lookup |
| `Monthly_Budget_Wide.xlsx` | Approved department budgets, one column per month | Clean, but wrong shape |

---

## Part A — Clean the Attendance files
Open each `Attendance_*.csv` in Power Query and fix, at minimum:
- `EmployeeID` has inconsistent casing and stray spaces — standardize it (this
  matters a lot later: your Merge in Part C will silently drop rows if IDs don't
  match exactly).
- `Date` is in several different formats within the *same* column — get it into a
  proper Date type.
- `Time In` / `Time Out` are inconsistent (12h/24h, missing values, `.` instead of
  `:`) — decide what you actually need for this project (hint: you may not need to
  fully standardize both if `Status` and `Hours Worked` already give you what the
  reports need).
- `Status` has multiple spellings for the same thing (e.g. Present appears at least
  5 different ways) — standardize into a fixed set of categories.
- `Hours Worked` mixes text and numbers (`"8 hrs"`, `"7.5"`, `"-"`) — clean into a
  proper numeric column.
- There are blank rows and a junk footer row ("Report generated on...") in every
  file — remove them.
- There are duplicate rows — remove them.

**Business question to answer along the way:** how many duplicate and blank rows
did each month contain before cleaning? (Worth noting in your report as a data
quality observation.)

## Part B — Append
Once all three months are individually clean and share identical column
structures, **append** them into a single `Master_Attendance` table covering
Jan–Mar.

*(If Power Query won't append cleanly, that's a signal a column name, type, or
structure didn't match across the three files — go back and check.)*

## Part C — Merge (enrich with master data)
`Master_Attendance` only has `EmployeeID` — it doesn't tell you which department or
branch someone belongs to. **Merge** it with `Employee_Master` on `EmployeeID` to
bring in `Department`, `Branch`, and `Designation`.

Do the same for the (cleaned) payroll data later in Part E — it will also need
`Department` from `Employee_Master`.

## Part D — Add Columns (Attendance side)
On the merged attendance table, add:
- A **Month** column (text or date, e.g. "Jan-2026") extracted from `Date`, since
  your final report needs monthly totals, not daily rows.
- A **Late Flag** column: mark `TRUE` if `Time In` is after 9:30 AM on days marked
  Present (only attempt this if you kept Time In in a usable form in Part A).

## Part E — Clean & Prep Payroll
`Payroll_Raw.csv` needs the same kind of treatment as the attendance files:
- `EmployeeID` casing/spacing.
- `Month` is written in at least 4 different formats — standardize it so it can
  later be compared with the Budget file's months.
- `Basic Pay`, `Allowance`, `Deduction`, `Bonus` are stored as text with currency
  symbols (`₹`, `Rs.`) and thousands separators — clean into numeric columns.
  Blank/`NA`/`-` deduction and bonus values should be treated as 0.
- Remove duplicate rows.

Then:
- **Merge** with `Employee_Master` to bring in `Department`.
- **Add Column**: `Net Pay = Basic Pay + Allowance + Bonus − Deduction`.

## Part F — Group By
Produce two grouped/aggregated tables:
1. From the attendance side: **Total Present Days per Department per Month**.
2. From the payroll side: **Total Net Pay per Department per Month**.

## Part G — Unpivot
`Monthly_Budget_Wide.xlsx` has one column per month (`Jan_Budget`, `Feb_Budget`,
`Mar_Budget`), which is fine for a printed report but useless for comparing against
your grouped payroll data. **Unpivot** the month columns so you get one row per
Department per Month with a `Budget Amount` column.

## Part H — Merge (Actual vs Budget) + Add Column
**Merge** your Part F payroll-by-department-by-month table with the unpivoted
budget table (Part G) on Department + Month. **Add a column**: `Variance = Net Pay
− Budget Amount`.

## Part I — Pivot (final report)
Take your Variance table and **pivot** it so that:
- Rows = Department
- Columns = Month
- Values = Variance (or Net Pay, your choice — state which you used)

This is your final "Budget vs Actual" matrix, ready to hand to HR leadership.

---

## Deliverables
1. The `.pbix` / Power Query workbook with all applied steps visible in each
   query's Applied Steps pane (don't delete your step history — that's how your
   work gets evaluated).
2. The final pivoted Budget vs Actual table.
3. The Attendance Summary table (present days per department per month).
4. A short note (5–6 lines) on the data quality issues you found and how you
   handled each one.

## A note on order
You'll notice you *can't* skip ahead — e.g. you can't Merge with `Employee_Master`
until `EmployeeID` is clean in both tables, and you can't Group By month until
`Month`/`Date` is a real, standardized value. That's intentional. In real BI work,
cleaning almost always gates everything downstream.
