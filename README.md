# 💼 Employee Payroll & Workforce Dashboard

An interactive **Power BI** dashboard that analyzes employee payroll cost, salary distribution, and workforce composition across departments, locations, designations, and gender — built on a star-schema data model.

![Dashboard Page 1 - Overview](https://github.com/DRasool7013/Employee-Payroll-Workforce-Dashboard/blob/main/Dashboard-Page1_Overview.png)

---

## 📌 Project Overview

This project turns raw employee and payroll records into a two-page executive dashboard that answers key HR and finance questions: where payroll cost is concentrated, how it trends over time, how pay compares across roles and departments, and how the workforce is composed by gender, location, and tenure.

| | |
|---|---|
| **Tool** | Power BI Desktop |
| **Data model** | Star schema — 1 fact table, 3 dimension tables |
| **Pages** | 2 (Payroll Overview, Workforce Insights) |
| **KPIs** | 5 |
| **Slicers** | 5 (synced across both pages) |
| **Charts** | 10 |

---

## 🗂️ Repository Structure

```
Employee-Payroll-Workforce-Dashboard/
├── Original Data/                 → source dataset(s) used to build the model
├── Images/
│   ├── Conceptual_Model.png       → high-level entity relationship
│   ├── Physical_Data_Model.png    → table/column-level schema from Power BI
│   ├── Dashboard_Page1_Overview.png
│   └── Dashboard_Page2_Workforce_Insights.png
├── Power BI File/
│   └── Employee_Payroll_Workforce_Dashboard.pbix
├── Documentation.md               → full write-up: goal, cleaning, DAX, insights
└── README.md                      → this file
```

---

## 🧬 Data Model

Star schema with one fact table and three dimension tables:

![Conceptual Model](https://github.com/DRasool7013/Employee-Payroll-Workforce-Dashboard/blob/main/Conceptual_Model.png)

| Table | Key Columns |
|---|---|
| **Fact Sales** (Payroll Fact) | SalaryLogID, EmployeeID, DeptID, SalaryDate, BasicSalary, Allowances, HRA, Bonus, Commission, Deductions, NetSalary |
| **Dim Employee** | EmpID, EmpName, Gender, Designation, DateOfJoining |
| **Dim Department** | DeptID, DeptName, Location |
| **Dim Calendar** | Date, MONTH, QUARTER, WEEKDAY, YEAR, Month Name |

Full physical schema with relationships (1-to-many, single direction, from each dimension into the fact table):

![Physical Data Model](https://github.com/DRasool7013/Employee-Payroll-Workforce-Dashboard/blob/main/Physical_Datamodel.png)

---

## 📊 KPI Cards (top row, both pages)

| KPI | DAX Measure |
|---|---|
| Total Net Salary | `SUM(FactSales[NetSalary])` |
| Total Employees | `DISTINCTCOUNT(FactSales[EmployeeID])` |
| Avg Net Salary | `AVERAGE(FactSales[NetSalary])` |
| Total Deductions | `SUM(FactSales[Deductions])` |
| Total Commission | `SUM(FactSales[Commission])` |

## 🎚️ Slicers (shared across both pages)

| Slicer | Source Field | Type |
|---|---|---|
| Date | `Dim Calendar[Date]` | Date slicer |
| DeptName | `Dim Department[DeptName]` | Dropdown/list |
| Location | `Dim Department[Location]` | Dropdown |
| Designation | `Dim Employee[Designation]` | Dropdown |
| Gender | `Dim Employee[Gender]` | Dropdown / toggle |

### How the slicers were added (Power BI Desktop)

1. Select **Report view**, click on an empty area of the canvas.
2. In the **Visualizations** pane, click the **Slicer** icon to insert a blank slicer.
3. Drag the target field (e.g. `Dim Calendar[Date]`) from the **Data** pane into the slicer's **Field** well.
4. Resize and position the slicer in the right-hand rail (this report stacks all 5 vertically).
5. Under **Format visual**, adjust the slicer style — this report uses **List** style for text fields and a **Between** date range for `Date`.
6. Repeat steps 2–5 for each of the remaining four fields (DeptName, Location, Designation, Gender).
7. To make slicers apply to *both* report pages: select each slicer → **Format** tab in the ribbon → **Sync slicers** → tick the pane icon for every page it should filter.
8. Test by selecting a value on Page 1 and confirming the filter carries over to Page 2.

---

## 📈 Charts & the Business Questions They Answer

**Page 1 — Payroll Overview**

| Chart | Business Question |
|---|---|
| Monthly Net Salary Trend (line) | How does total payroll cost trend month over month? |
| Total Net Salary by DeptName (donut) | Which department has the highest payroll cost? |
| Avg Net Salary by Designation (bar) | How does pay vary by role/designation? |
| Total Net Salary by Location & Gender (column) | How is payroll cost distributed geographically, and by gender? |
| Monthly Commission & Bonus (line) | What is the total bonus and commission paid each month? |

**Page 2 — Workforce Insights**

| Chart | Business Question |
|---|---|
| Employees by Gender (pie) | What is the gender-wise distribution of employees? |
| Headcount by Department (column) | How many employees are there in each department? |
| Total Net Salary by DeptName (column) | What share of total payroll cost does each department contribute? |
| Average Net Salary by Department & Gender (column) | How does average salary differ by department and gender? |
| Employees Joined by Year (line) | How has headcount grown — total employees joined by year? |

![Dashboard Page 2 - Workforce Insights](https://github.com/DRasool7013/Employee-Payroll-Workforce-Dashboard/blob/main/Dashboard-Page2_Workforce_Insights.png)

---

See **[Documentation.md](Documentation.md)** for the full write-up: project goal, dataset description, data-cleaning steps, DAX measures, final results, and insights.

---

## 👤 Author

Built as a Power BI portfolio project. Feedback and suggestions welcome via Issues/PRs.
