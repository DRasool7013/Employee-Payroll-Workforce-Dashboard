# Documentation — Employee Payroll & Workforce Dashboard

## 1. Project Aim / Goal

The goal of this project is to give HR and finance stakeholders a single, interactive view of payroll cost and workforce composition — so they can see where salary spend is concentrated, how it moves month to month, and how the workforce breaks down by department, location, role, and gender, without pulling manual reports.

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

The source data models a company's payroll system as a star schema with one transactional fact table and three descriptive dimension tables.

| Table | Role | Fields |
|---|---|---|
| **Fact Sales** (Payroll Fact) | Grain: one row per salary log/payroll run per employee | `SalaryLogID`, `EmployeeID`, `DeptID`, `SalaryDate`, `BasicSalary`, `Allowances`, `HRA`, `Bonus`, `Commission`, `Deductions`, `NetSalary` |
| **Dim Employee** | One row per employee | `EmpID`, `EmpName`, `Gender`, `Designation`, `DateOfJoining` |
| **Dim Department** | One row per department | `DeptID`, `DeptName`, `Location` |
| **Dim Calendar** | Standard date table | `Date`, `MONTH`, `QUARTER`, `WEEKDAY`, `YEAR`, `Month Name` |

Relationships: `Fact Sales` connects to `Dim Employee` on `EmployeeID`, to `Dim Department` on `DeptID`, and to `Dim Calendar` on `SalaryDate` → `Date` — all single-direction, one (dimension) to many (fact).

![Conceptual Model](https://github.com/DRasool7013/Employee-Payroll-Workforce-Dashboard/blob/main/Conceptual_Model.png)

## 3. Cleaning & Modeling Process

- Split the flat payroll export into a fact table (transactional salary components) and three dimension tables (Employee, Department, Calendar) to remove repeated attribute values and support a proper star schema.
- Built a dedicated `Dim Calendar` table with `Date`, `MONTH`, `QUARTER`, `WEEKDAY`, `YEAR`, and `Month Name` columns to support month-over-month and year-over-year trend analysis.
- Set relationship cardinality to one-to-many from each dimension table into `Fact Sales`, with single-direction filtering.
- Standardized categorical fields (`DeptName`, `Location`, `Gender`, `Designation`) for consistent slicer values.
- Verified `EmployeeID` and `DeptID` are populated on every fact row so no rows are lost on the join.

## 4. Measures (DAX)

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

> Note: the built dashboard's 5th KPI card uses **Total Commission** rather than **MoM Net Salary Change %**, to keep all five cards as simple point-in-time totals; the MoM measure is still defined in the model and available for a trend-style KPI visual if needed.

## 5. Final Result

Two report pages, five shared slicers, five KPI cards, and ten charts — detailed in full in the [README](README.md#-charts--the-business-questions-they-answer).

![Dashboard Page 1](https://github.com/DRasool7013/Employee-Payroll-Workforce-Dashboard/blob/main/Dashboard-Page1_Overview.png)
![Dashboard Page 2](Images/Dashboard_Page2_Workforce_Insights.png)

## 6. Outcomes / Insights

Based on the current dataset (13 employees, 4 departments, 4 locations):

1. **Overall payroll**: Total Net Salary paid across the period is **$30.6M**, across **13 employees**, for an average net salary of **$65.5K** per employee. Total deductions are **$931.8K** and total commission is **$3.5M**.
2. **Payroll trend**: Monthly Net Salary Trend fluctuates in the ~$2.50M–$2.60M/month range, with no sustained upward or downward trend — payroll cost is relatively stable month to month.
3. **Department cost concentration**: HR carries the largest share of total net salary (**$8.7M, ~28%**), followed by Research (**$7.7M, ~25%**), Sales (**$7.4M, ~24%**), and Accounting (**$6.9M, ~22%**) — a fairly even split across all four departments, with HR as the largest.
4. **Pay by role**: Managers and Presidents earn the highest average net salary (**~$72K**), followed by Salesman (**~$67K**), Analyst (**~$62K**), and Clerk (**~$60K**) — roughly a $12K spread between the highest- and lowest-paid designations.
5. **Geographic distribution**: Net salary is highest in New York (**$7.7M**) and Dallas (**$6.9M**), with San Francisco and Chicago lower; Chicago and New York both show a split between male and female pay within the location.
6. **Gender distribution**: The workforce is **76.9% male (10)** and **23.1% female (3)** — a significant gender imbalance worth flagging for workforce planning.
7. **Bonus & commission**: Both commission and bonus fluctuate month to month without a clear seasonal pattern, generally trading places between roughly $0.2M and $0.35M per month.
8. **Headcount by department**: HR has the most employees (4), with Accounting, Research, and Sales each close behind (~3 each) — headcount is fairly evenly distributed, consistent with the near-even payroll-cost split.
9. **Average salary by department & gender**: Accounting and Sales show the highest average salaries (**~$72K**), with a visible male/female pay gap in most departments — worth a closer look for pay-equity review.
10. **Hiring trend**: Employees Joined by Year peaks around **2022 (4 joiners)**, with lower and flatter joining activity before 2020 and after 2022 — most of the current workforce was hired in a single hiring wave rather than steadily over time.

## 7. Next Steps

- Add a **pay-equity view** isolating the male/female average salary gap per department and designation.
- Extend `Dim Calendar` with a rolling 12-month window so the MoM Net Salary Change % KPI can be surfaced as a trend card.
- Bring in headcount attrition data (if available) to pair with `Employees Joined by Year` for a full hiring-vs-attrition picture.
