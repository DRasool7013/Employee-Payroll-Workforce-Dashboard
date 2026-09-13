# Documentation — Employee Payroll & Workforce Dashboard

## 1. Project Aim / Goal

The goal of this project is to give HR and finance stakeholders a single, interactive view of payroll cost and workforce composition — so they can see where salary spend is concentrated, how it moves month to month, and how the workforce breaks down by department, location, role, and gender.

Specifically, the dashboard was built to answer:

1. How does total payroll cost trend month over month?
2. Which department has the highest payroll cost?
3. How does pay vary by role/designation?
4. How is payroll cost and headcount distributed geographically?
5. What is the gender-wise distribution of employees?
6. How many employees are there in each department?
7. What is the total bonus and commission paid each month?
8. What share of total payroll cost does each department contribute?
9. How does average salary differ by department and gender?
10. How has headcount grown — total employees joined by year?

## 2. Dataset Description

Source: ![Original Data/Employee-Payroll_dataset.xlsx]([Images/Physical_Datamodel.png](https://github.com/DRasool7013/Employee-Payroll-Workforce-Dashboard/blob/main/Employee-Payroll_dataset.xlsx), a raw workbook with three sheets:

| Sheet | Rows (excl. header) | Columns |
|---|---|---|
| **Department** | 5 | DeptID, DeptName, Location |
| **Salary** | 504 | SalaryLogID, EmployeeID, DeptID, SalaryDate, BasicSalary, HRA, Allowances, Commission, Bonus, Deductions, NetSalary |
| **Employee** | 50 (14 distinct EmpIDs, each repeated 3–4×) | EmpID, EmpName, Designation, Gender, DateOfJoining |

The raw data is intentionally messy — typical of an unclean HR export:

- **Department/Location** has typos and inconsistent casing: `CHICGO`, `san fransisco`, `dallas`, `BOSTON ` (trailing space), `NEW YORK`.
- **Employee** has duplicate `EmpID` rows (each of the 14 real employees appears 3–4 times) with **conflicting Gender and DateOfJoining values across duplicates** for the same person.
- **Designation** has multiple misspelled/inconsistent variants per role, e.g. Clerk appears as `CLERK`, `CLRK`, `clerck`; Manager as `MANAGER`, `MANGER`, `MANEGER`; Salesman as `SALESMAN`, `SALESMN`, `salesman`; Analyst as `ANALYST`, `ANLYST`; President as `PRESIDNT`.
- **SalaryDate** is stored as text in `dd-mm-yyyy` format rather than a proper date type.
- Of the 50 Employee rows in the raw sheet, only **14 unique EmpIDs** actually have matching Salary records — the rest of the Employee sheet is redundant/duplicate rows for those same 14 people.

## 3. Cleaning & Modeling Process

- **Deduplicated `Employee`** down from 50 rows to 14 unique `EmpID` records, keeping one canonical Gender/Designation/DateOfJoining per employee.
- **Standardized `Designation`** by mapping every misspelled variant to one of five canonical values: `Clerk`, `Salesman`, `Manager`, `Analyst`, `President`.
- **Cleaned `Location`**, fixing typos and casing: `CHICGO` → `Chicago`, `san fransisco` → `San Francisco`, `dallas` → `Dallas`, `BOSTON ` (trailing space) → `Boston`, `NEW YORK` → `New York`.
- **Converted `SalaryDate`** from text (`dd-mm-yyyy`) to a proper `Date` type, then built a dedicated `Dim Calendar` table with `Date`, `MONTH`, `QUARTER`, `WEEKDAY`, `YEAR`, and `Month Name` to support month-over-month and year-over-year analysis.
- **Excluded the Operations department** (`DeptID 40`, Boston) from the active model — its only associated employee record was dropped during deduplication, so no payroll rows remain against it; this is why the final report shows 13 employees across 4 departments (Hr, Accounting, Sales, Research) rather than the 14 employee IDs and 5 departments present in the raw file. `Boston` may still appear as an available (empty) option in the Location slicer since it comes from the dimension table.
- Split the cleaned data into a fact table (transactional salary components) and three dimension tables (Employee, Department, Calendar) for a proper star schema, with one-to-many, single-direction relationships from each dimension into `Fact Sales`.

## 4. Data Model

![Conceptual Model]()

| Table | Role | Fields |
|---|---|---|
| **Fact Sales** (Payroll Fact) | Grain: one row per salary log per employee | `SalaryLogID`, `EmployeeID`, `DeptID`, `SalaryDate`, `BasicSalary`, `HRA`, `Allowances`, `Commission`, `Bonus`, `Deductions`, `NetSalary` |
| **Dim Employee** | One row per employee (post-dedup) | `EmpID`, `EmpName`, `Designation`, `Gender`, `DateOfJoining` |
| **Dim Department** | One row per department | `DeptID`, `DeptName`, `Location` |
| **Dim Calendar** | Standard date table | `Date`, `MONTH`, `QUARTER`, `WEEKDAY`, `YEAR`, `Month Name` |

![Physical Data Model]([Images/Physical_Datamodel.png](https://github.com/DRasool7013/Employee-Payroll-Workforce-Dashboard/blob/main/Physical_Datamodel.png))

## 5. Measures (DAX)

```DAX
Total Basic Salary   = SUM(FactSales[BasicSalary])
Total Allowances     = SUM(FactSales[Allowances])
Total Bonus           = SUM(FactSales[Bonus])
Total Commission     = SUM(FactSales[Commission])
Total Deductions     = SUM(FactSales[Deductions])
Total HRA            = SUM(FactSales[HRA])
Total Net Salary     = SUM(FactSales[NetSalary])
Total Employees      = DISTINCTCOUNT(FactSales[EmployeeID])
Total Payroll Runs   = DISTINCTCOUNT(FactSales[SalaryLogID])
Avg Net Salary       = AVERAGE(FactSales[NetSalary])

Deduction % of Gross =
DIVIDE(
    [Total Deductions],
    [Total Basic Salary] + [Total Allowances] + [Total Bonus] + [Total Commission] + [Total HRA],
    0
)

MoM Net Salary Change % =
VAR CurrentM = [Total Net Salary]
VAR PrevM = CALCULATE([Total Net Salary], DATEADD(DimCalender[Date], -1, MONTH))
RETURN DIVIDE(CurrentM - PrevM, PrevM, 0)
```

> Note: the built dashboard's 5th KPI card uses **Total Commission** rather than **MoM Net Salary Change %**, keeping all five cards as simple point-in-time totals; the MoM measure is still defined in the model and available for a trend-style KPI visual if needed.

## 6. Final Result

Two report pages, five shared slicers, five KPI cards, and ten charts — see the [README](README.md#-charts--the-business-questions-they-answer) for the full chart-to-question mapping.

![Dashboard Page 1](Images/Dashboard-Page1_Overview.png)
![Dashboard Page 2](Images/Dashboard-Page2_Workforce_Insights.png)

## 7. Outcomes / Insights

Based on the cleaned dataset (13 employees, 4 departments, 4 active locations):

1. **Overall payroll**: Total Net Salary paid is **$30.6M** across **13 employees**, for an average net salary of **$65.5K** per employee. Total deductions are **$931.8K** and total commission is **$3.5M**.
2. **Payroll trend**: Monthly Net Salary Trend fluctuates in the ~$2.50M–$2.60M/month range with no sustained upward or downward trend — payroll cost is relatively stable month to month.
3. **Department cost concentration**: HR carries the largest share of total net salary (**$8.7M, ~28%**), followed by Research (**$7.7M, ~25%**), Sales (**$7.4M, ~24%**), and Accounting (**$6.9M, ~22%**) — a fairly even split, with HR the largest.
4. **Pay by role**: Managers and Presidents earn the highest average net salary (**~$72K**), followed by Salesman (**~$67K**), Analyst (**~$62K**), and Clerk (**~$60K**) — roughly a $12K spread between the highest- and lowest-paid designations.
5. **Geographic distribution**: Net salary is highest in New York (**$7.7M**) and Dallas (**$6.9M**), with San Francisco and Chicago lower; Chicago and New York both show a male/female split within the location.
6. **Gender distribution**: The workforce is **76.9% male (10)** and **23.1% female (3)** — a significant gender imbalance worth flagging for workforce planning.
7. **Bonus & commission**: Both fluctuate month to month with no clear seasonal pattern, generally trading places between roughly $0.2M and $0.35M per month.
8. **Headcount by department**: HR has the most employees (4), with Accounting, Research, and Sales close behind (~3 each) — headcount is fairly evenly distributed, consistent with the near-even payroll-cost split.
9. **Average salary by department & gender**: Accounting and Sales show the highest average salaries (**~$72K**), with a visible male/female pay gap in most departments — worth a closer look for pay-equity review.
10. **Hiring trend**: Employees Joined by Year peaks around **2022 (4 joiners)**, with lower and flatter joining activity before 2020 and after 2022 — most of the current workforce was hired in a single hiring wave rather than steadily over time.

## 8. Next Steps

- Add a **pay-equity view** isolating the male/female average salary gap per department and designation.
- Re-examine the Employee sheet's duplicate records at the source to confirm which Gender/DateOfJoining value is correct per employee, rather than picking one canonical row.
- Decide whether to keep or fully remove the empty **Operations/Boston** department from the report and its slicers, since it currently has no active payroll data.
- Extend `Dim Calendar` with a rolling 12-month window so the **MoM Net Salary Change %** KPI can be surfaced as a trend card.
