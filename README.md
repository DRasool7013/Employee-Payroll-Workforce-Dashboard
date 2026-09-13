# 💼 Employee Payroll & Workforce Dashboard

An interactive **Power BI** dashboard analyzing employee payroll cost and workforce composition — department, location, designation, and gender — built on a cleaned star-schema data model.

![Dashboard Page 1 - Overview](https://github.com/DRasool7013/Employee-Payroll-Workforce-Dashboard/blob/main/Dashboard-Page1_Overview.png)

---

## 📌 Project Overview

| | |
|---|---|
| **Tool** | Power BI Desktop |
| **Source data** | `Employee-Payroll_dataset.xlsx` — 3 sheets (Department, Salary, Employee) |
| **Data model** | Star schema — 1 fact table, 3 dimension tables |
| **Pages** | 2 (Payroll Overview, Workforce Insights) |
| **KPIs** | 5 |
| **Slicers** | 5 (synced across both pages) |
| **Charts** | 10 |

---

## 🗂️ Repository Structure — All Files in This Project

```
Employee-Payroll-Workforce-Dashboard/
│
├── Original Data/
│   └── Employee-Payroll_dataset.xlsx      → raw source workbook (Department, Salary, Employee sheets)
│
├── Images/
│   ├── Conceptual_Model.png               → high-level entity relationship
│   ├── Physical_Datamodel.png             → table/column-level schema from Power BI
│   ├── Dashboard-Page1_Overview.png        → Page 1 screenshot
│   └── Dashboard-Page2_Workforce_Insights.png → Page 2 screenshot
│
├── Power BI File/
│   └── Employee-Payroll_&_Workforce_Dashboard.pbix   → the working Power BI report
│
├── Documentation.md                        → project aim, dataset, cleaning process, DAX, insights
└── README.md                               → this file
```

---

## 🧬 Data Model

Star schema with one fact table and three dimension tables:

![Conceptual Model](Images/Conceptual_Model.png)

| Table | Key Columns |
|---|---|
| **Fact Sales** (Payroll Fact) | SalaryLogID, EmployeeID, DeptID, SalaryDate, BasicSalary, HRA, Allowances, Commission, Bonus, Deductions, NetSalary |
| **Dim Employee** | EmpID, EmpName, Designation, Gender, DateOfJoining |
| **Dim Department** | DeptID, DeptName, Location |
| **Dim Calendar** | Date, MONTH, QUARTER, WEEKDAY, YEAR, Month Name |

Full physical schema with relationships (1-to-many, single direction, from each dimension into the fact table):

![Physical Data Model](Images/Physical_Datamodel.png)

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

### Steps to Add the Slicers (Power BI Desktop)

1. Go to **Report view**, click an empty area of the canvas.
2. In the **Visualizations** pane, click the **Slicer** icon to insert a blank slicer.
3. Drag the target field (e.g. `Dim Calendar[Date]`) from the **Data** pane into the slicer's **Field** well.
4. Resize and position the slicer in the right-hand rail — this report stacks all five vertically: Date, DeptName, Location, Designation, Gender.
5. Under **Format visual**, set the slicer style — **Between** date range for `Date`, **List** style for the four category fields.
6. Repeat steps 2–5 for each remaining field.
7. To make slicers apply to **both** report pages: select each slicer → **Format** tab on the ribbon → **Sync slicers** → tick the page(s) it should also filter.
8. Test by selecting a value on Page 1 and confirming the filter carries through to Page 2.

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
| Employees Joined by Year (line) | How has headcount grown — employees joined by year? |

![Dashboard Page 2 - Workforce Insights](Images/Dashboard-Page2_Workforce_Insights.png)

---

## 🚀 How to Use This Repo

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/Employee-Payroll-Workforce-Dashboard.git

# 2. Open the .pbix file
Open "Power BI File/Employee-Payroll_&_Workforce_Dashboard.pbix" in Power BI Desktop

# 3. Point Power Query at Original Data/Employee-Payroll_dataset.xlsx and refresh
Home → Refresh
```

See **[Documentation.md](Documentation.md)** for the full write-up: project goal, dataset description, data-cleaning steps, DAX measures, final results, and insights.

---

## 🛠️ Publishing This Repo to GitHub

```bash
cd Employee-Payroll-Workforce-Dashboard
git init
git add .
git commit -m "Initial commit: Employee Payroll & Workforce Dashboard"
git branch -M main
git remote add origin https://github.com/<your-username>/Employee-Payroll-Workforce-Dashboard.git
git push -u origin main
```

> 💡 Tip: add a `.gitignore` for local Power BI temp/cache files if you rebuild the `.pbix` (e.g. `*.tmp`, `~$*`).

---

## 👤 Author

Built as a Power BI portfolio project. Feedback and suggestions welcome via Issues/PRs.
