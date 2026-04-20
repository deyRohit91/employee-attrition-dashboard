# 🧑‍💼 Employee Attrition Analysis

> **End-to-end HR analytics project** — data cleaning, exploratory analysis, and attrition driver identification on a 47,566-employee dataset.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Dataset](#-dataset)
- [Data Cleaning](#-data-cleaning)
- [Key Findings](#-key-findings)
- [Attrition by Department](#-attrition-by-department)
- [Risk Factors](#-risk-factors)
- [Exit Reasons](#-exit-reasons)
- [Satisfaction & Performance](#-satisfaction--performance)
- [Recommendations](#-recommendations)
- [Project Structure](#-project-structure)
- [How to Run](#-how-to-run)
- [Technologies Used](#-technologies-used)

---

## 📌 Project Overview

This project performs a full end-to-end analysis of employee attrition using HR data from a simulated enterprise environment. The goal is to:

1. **Clean** real-world messy HR data (12 documented issues resolved)
2. **Explore** attrition patterns across departments, demographics, and work conditions
3. **Identify** the key drivers that predict whether an employee will leave
4. **Recommend** data-backed retention strategies

The project is structured as a complete analytics workflow that a real data team would present to HR leadership.

---

## 📂 Dataset

| Property | Value |
|---|---|
| **File** | `employee_attrition_final.csv` |
| **Records** | 47,566 employees |
| **Features** | 46 columns |
| **Target variable** | `Attrition` (Yes / No) |
| **Hire period** | 2008 – 2023 |
| **Departments** | 10 |
| **Data completeness** | 100% (after cleaning) |

### Column Groups

<details>
<summary><strong>Identity (10 columns)</strong></summary>

| Column | Type | Description |
|---|---|---|
| `EmployeeID` | string | Unique employee identifier (format: EMP000001) |
| `Age` | int | Employee age in years |
| `Gender` | string | Male / Female / Non-Binary |
| `MaritalStatus` | string | Single / Married / Divorced |
| `Education` | string | Highest education level |
| `EducationField` | string | Field of study |
| `Region` | string | Geographic region (North/South/East/West/Central) |
| `Department` | string | Business department |
| `JobRole` | string | Specific job title |
| `EmploymentType` | string | Full-Time / Part-Time / Contract |

</details>

<details>
<summary><strong>Tenure & Experience (7 columns)</strong></summary>

| Column | Type | Description |
|---|---|---|
| `HireDate` | datetime | Date the employee joined |
| `YearsAtCompany` | float | Total years at the company |
| `YearsInCurrentRole` | float | Years in current role |
| `YearsSinceLastPromotion` | float | Years since last promotion |
| `TotalWorkingYears` | float | Total career experience |
| `NumCompaniesBefore` | int | Number of prior employers |
| `Tenure` | float | Exact tenure in years (calculated from HireDate) |

</details>

<details>
<summary><strong>Compensation (6 columns)</strong></summary>

| Column | Type | Description |
|---|---|---|
| `MonthlyIncome` | float | Monthly salary in USD |
| `HourlyRate` | float | Derived hourly rate |
| `DailyRate` | float | Derived daily rate |
| `PercentSalaryHike` | float | Last salary increase % |
| `StockOptionLevel` | int | Stock option level (0–3) |
| `HireYear` / `HireMonth` | int | Extracted from HireDate |

</details>

<details>
<summary><strong>Satisfaction & Performance (7 columns)</strong></summary>

| Column | Type | Description |
|---|---|---|
| `JobSatisfaction` | int | Score 1–5 |
| `EnvironmentSatisfaction` | int | Score 1–5 |
| `RelationshipSatisfaction` | int | Score 1–5 |
| `ManagerRating` | int | Score 1–5 |
| `PerformanceRating` | string | Low / Average / High / Excellent |
| `LastPerformanceScore` | float | Numeric score 1.0–5.0 |
| `WorkLifeBalance` | string | Poor / Fair / Good / Excellent |

</details>

<details>
<summary><strong>Work Conditions & Behavioral (10 columns)</strong></summary>

| Column | Type | Description |
|---|---|---|
| `OverTime` | string | Yes / No |
| `BusinessTravel` | string | No Travel / Rarely / Frequently |
| `DistanceFromHome_km` | int | Commute distance |
| `RemoteWorkPercent` | int | % of time remote (0/25/50/75/100) |
| `NumProjectsHandled` | int | Active project count |
| `TrainingTimesLastYear` | int | Training sessions attended |
| `AbsenteeismDays` | int | Days absent this year |
| `GrievancesFiled` | int | HR grievances raised |
| `AwardsWon` | int | Recognition awards |
| `TeamSize` / `NumDirectReports` | int | Team structure metrics |

</details>

<details>
<summary><strong>Target Variables (2 columns)</strong></summary>

| Column | Type | Description |
|---|---|---|
| `Attrition` | string | **Target** — Yes (left) / No (active) |
| `ExitReason` | string | Reason for leaving (or "Not Applicable" for active employees) |

</details>

---

## 🧹 Data Cleaning

The original dataset (`employee_attrition.csv`) contained **12 documented data quality issues** that were resolved before analysis.

### Issues Fixed

| # | Issue | Rows Affected | Fix Applied |
|---|---|---|---|
| 1 | Education `str.title()` bug — `"Bachelor'S Degree"`, `"Phd"` | 40,864 rows | Explicit replace dictionary |
| 2 | JobRole acronym casing — `"Hr Analyst"`, `"Cfo"`, `"It Support"` | 8,747 rows | Explicit replace dictionary |
| 3 | Duplicate EmployeeIDs | 485 rows | `drop_duplicates(subset='EmployeeID')` |
| 4 | EmployeeID format changed from `EMP` to `Emp` | All rows | Regex restore to `EMP` prefix |
| 5 | HireDate stored as string instead of datetime | All rows | `pd.to_datetime()` |
| 6 | Tenure calculated using year 2026 (off by +2 years) | All rows | Recalculated from `pd.Timestamp('2024-01-01')` |
| 7 | `YearsInCurrentRole > YearsAtCompany` (impossible) | 1,298 rows | Capped at `YearsAtCompany` |
| 8 | `TotalWorkingYears < YearsAtCompany` (impossible) | 386 rows | Set equal to `YearsAtCompany` |
| 9 | `YearsSinceLastPromotion > YearsAtCompany` (impossible) | 364 rows | Capped at `YearsAtCompany` |
| 10 | `LastPerformanceScore` and `DistanceFromHome_km` still missing | 479 + 382 NaN | Median imputation |
| 11 | 6 float columns that should be integers | Multiple columns | Cast to `int64` |
| 12 | `HourlyRate` / `DailyRate` out of sync after income correction | 1,938 rows | Recomputed from `MonthlyIncome` |

### Data Quality — Before vs After

```
                          BEFORE      AFTER
Total rows                50,000     47,566
Total columns                 43         46
Missing values             8,092          0
Duplicate EmployeeIDs        485          0
Logic errors (years)       2,048          0
Attrition encoding variants      8          2  (Yes/No only)
Education correct values   False       True
HireDate as datetime       False       True
Tenure accurate            False       True  (uses 2024 ref)
Data completeness          98.1%      100.0%
```

---

## 📊 Key Findings

### Overall Attrition

```
Total Employees  :  47,566
Active           :  30,976  (65.1%)
Attrited         :  16,590  (34.9%)
Avg Monthly Salary:  $9,065
Avg Tenure       :  8.0 years
Avg Job Satisfaction: 2.99 / 5
```

> ⚠️ **The overall attrition rate of 34.9% means approximately 1 in 3 employees has left the organisation.**

---

## 🏢 Attrition by Department

All departments sit within a narrow 1pp band — confirming this is an **organisation-wide structural issue**, not a department-specific problem.

| Department | Total | Attrited | Rate |
|---|---|---|---|
| Marketing | 4,801 | 1,693 | 🔴 35.3% |
| HR | 4,846 | 1,709 | 🔴 35.3% |
| Finance | 4,782 | 1,681 | 🔴 35.2% |
| Legal | 4,648 | 1,630 | 🟠 35.1% |
| R&D | 4,690 | 1,640 | 🟠 35.0% |
| Customer Support | 4,777 | 1,673 | 🟠 35.0% |
| Engineering | 4,843 | 1,686 | 🟡 34.8% |
| Sales | 4,726 | 1,633 | 🟡 34.6% |
| Operations | 4,742 | 1,630 | 🟢 34.4% |
| IT | 4,711 | 1,615 | 🟢 34.3% |

---

## ⚠️ Risk Factors

Ranked by size of impact (percentage-point difference vs baseline):

### 1. Overtime — Biggest Single Driver (+10.1pp)

| Group | Attrition Rate |
|---|---|
| Works Overtime | **41.5%** 🔴 |
| No Overtime | 31.4% 🟢 |
| **Difference** | **+10.1 percentage points** |

Overtime is the **largest single predictor of attrition** in this dataset. Employees working overtime leave at a rate 10 percentage points higher than those who don't.

### 2. Work-Life Balance (+6.6pp)

| Rating | Attrition Rate |
|---|---|
| Poor | **40.8%** 🔴 |
| Fair | 34.2% |
| Good | 34.1% 🟢 |
| Excellent | 34.6% |

Closely correlated with overtime — poor WLB and overtime together create a compounding risk effect.

### 3. Business Travel (+5.4pp)

| Travel Frequency | Attrition Rate |
|---|---|
| Frequently | **39.1%** 🔴 |
| No Travel | 34.1% |
| Rarely | 33.7% 🟢 |

### 4. Marital Status (+3.9pp)

| Status | Attrition Rate |
|---|---|
| Single | **37.5%** 🔴 |
| Married | 33.7% |
| Divorced | 33.5% 🟢 |

Single employees face lower switching costs and greater geographic flexibility.

### 5. Tenure — Long-Tenured Staff at Risk (+1.8pp)

| Tenure Band | Attrition Rate |
|---|---|
| < 1 year | 33.8% |
| 1–3 years | 33.9% |
| 3–6 years | 34.8% |
| 6–10 years | 34.6% |
| **10+ years** | **35.6%** 🔴 |

Long-tenured employees show the highest attrition — likely driven by career plateau and lack of promotion opportunities.

### Performance — High Performers Not Retained Better

| Rating | Attrition Rate |
|---|---|
| Low | 39.8% 🔴 |
| Average | 34.3% |
| High | 34.8% |
| Excellent | 34.8% |

> ❗ **Critical finding:** High and Excellent performers leave at virtually the same rate as Average performers. Top talent is not being retained better than average talent — a gap in current retention strategy.

---

## 🚪 Exit Reasons

Exit reason data available for 16,388 of 16,590 attrited employees (98.8% coverage).

| Rank | Exit Reason | Count | % of Exits |
|---|---|---|---|
| 1 | Work-Life Balance | 1,687 | 10.3% |
| 2 | Health Issues | 1,665 | 10.2% |
| 3 | Company Culture | 1,646 | 10.0% |
| 4 | Personal Reasons | 1,645 | 10.0% |
| 5 | Job Insecurity | 1,644 | 10.0% |
| 6 | Relocation | 1,642 | 10.0% |
| 7 | Manager Conflict | 1,633 | 10.0% |
| 8 | Career Growth | 1,618 | 9.9% |
| 9 | Better Opportunity | 1,608 | 9.8% |
| 10 | Higher Salary | 1,600 | 9.8% |

> 💡 **Key insight:** All 10 exit reasons are at approximately 10% each — no single dominant cause. This indicates employees leave due to an **accumulation of dissatisfactions**, not one fixable issue. A broad-based retention strategy is required.

---

## 😐 Satisfaction & Performance

### Job Satisfaction vs Attrition

| Score | Attrited | Active | Attrition Rate |
|---|---|---|---|
| 1 — Very Low | 3,970 | 5,420 | **42.3%** 🔴 |
| 2 — Low | 3,860 | 5,337 | **42.0%** 🔴 |
| 3 — Neutral | 3,126 | 7,429 | 29.6% 🟢 |
| 4 — High | 2,813 | 6,470 | 30.3% 🟢 |
| 5 — Very High | 2,821 | 6,320 | 30.9% 🟢 |

Employees with satisfaction scores of 1–2 leave at over 42% — more than 12 percentage points higher than those scoring 3–5.

---

## 💡 Recommendations

### Priority 1 — Immediate

| # | Action | Evidence |
|---|---|---|
| R1 | Conduct an overtime audit across all departments; redesign workloads or add headcount for structurally overloaded roles | Overtime → 41.5% attrition vs 31.4% — a 10.1pp gap |
| R2 | Launch a targeted Work-Life Balance pulse survey to the 4,816 employees who rated WLB as Poor | Poor WLB → 40.8% attrition — 5.9pp above average |

### Priority 2 — Short-Term (Next Quarter)

| # | Action | Evidence |
|---|---|---|
| R3 | Review business travel policy — set annual travel thresholds and increase virtual engagement options | Frequent travellers → 39.1% attrition, a 5.4pp premium |
| R4 | Develop a targeted retention programme for single employees: enhanced flexibility and career development | Single employees → 37.5% vs 33.5–33.7% for other groups |
| R5 | Investigate career plateau risk for 10+ year employees; review promotion rates for long-tenured staff | 10+ yr tenure → 35.6% attrition, the highest of all tenure bands |
| R6 | Create differentiated retention for High/Excellent performers: equity, accelerated pay reviews, internal mobility | High/Excellent → 34.8% attrition, nearly identical to Average at 34.3% |

### Priority 3 — Strategic (6 Months)

| # | Action | Evidence |
|---|---|---|
| R7 | Design a broad-based engagement programme addressing all top exit reasons simultaneously | All 10 exit reasons at ~10% each — no single fix will move the needle |
| R8 | Commission a manager effectiveness review and 360-degree feedback programme | Manager Conflict is the 7th most common exit reason |
| R9 | Benchmark salaries against market rates across all 10 departments | Salary + Better Opportunity = ~19.6% of exits combined |

---

## 📁 Project Structure

```
employee-attrition-analysis/
│
├── data/
│   ├── employee_attrition.csv           # Original raw dataset (50,000 rows)
│   └── employee_attrition_final.csv     # Cleaned dataset (47,566 rows, 46 cols)
│
├── notebooks/
│   ├── 01_data_cleaning.py              # 12-step cleaning pipeline
│   ├── 02_eda_analysis.py               # Exploratory data analysis
│   └── 03_attrition_drivers.py          # Risk factor & correlation analysis
│
├── reports/
│   ├── Attrition_Team_Report.docx       # Full report for team lead
│   └── DataCleaning_AuditReport.docx    # Data cleaning audit log
│
└── README.md                            # This file
```

---

## ▶️ How to Run

### Prerequisites

```bash
pip install pandas numpy
```

### Step 1 — Load the cleaned dataset

```python
import pandas as pd
import numpy as np

df = pd.read_csv('data/employee_attrition_final.csv')
print(f"Shape: {df.shape}")
print(f"Attrition rate: {(df['Attrition']=='Yes').mean()*100:.1f}%")
```

### Step 2 — Quick attrition breakdown

```python
# Overall rate
print(df['Attrition'].value_counts(normalize=True).mul(100).round(1))

# By Department
dept_rate = (df.groupby('Department')['Attrition']
               .apply(lambda x: (x=='Yes').mean()*100)
               .round(1)
               .sort_values(ascending=False))
print(dept_rate)
```

### Step 3 — Key risk factor analysis

```python
# Overtime impact
ot = df.groupby('OverTime')['Attrition'].apply(lambda x: (x=='Yes').mean()*100).round(1)
print("Overtime impact:", ot.to_dict())

# Work-life balance impact
wlb = df.groupby('WorkLifeBalance')['Attrition'].apply(lambda x: (x=='Yes').mean()*100).round(1)
print("WLB impact:", wlb.to_dict())

# Satisfaction vs attrition
sat = df.groupby('JobSatisfaction')['Attrition'].apply(lambda x: (x=='Yes').mean()*100).round(1)
print("Satisfaction impact:", sat.to_dict())
```

### Step 4 — Correlation with attrition (NumPy)

```python
df['Attrition_Binary'] = (df['Attrition'] == 'Yes').astype(int)

numeric_features = [
    'MonthlyIncome', 'JobSatisfaction', 'OverTime',
    'YearsSinceLastPromotion', 'AbsenteeismDays', 'Tenure'
]

correlations = {}
for feat in ['MonthlyIncome','JobSatisfaction','YearsSinceLastPromotion','AbsenteeismDays','Tenure']:
    x = df[feat].values.astype(float)
    y = df['Attrition_Binary'].values.astype(float)
    x_m = x - np.mean(x)
    y_m = y - np.mean(y)
    r = np.dot(x_m, y_m) / (np.sqrt(np.dot(x_m, x_m)) * np.sqrt(np.dot(y_m, y_m)) + 1e-9)
    correlations[feat] = round(r, 4)

for feat, r in sorted(correlations.items(), key=lambda x: abs(x[1]), reverse=True):
    direction = "increases attrition" if r > 0 else "reduces attrition"
    print(f"  {feat:<35} r={r:+.4f}  ({direction})")
```

---

## 🛠️ Technologies Used

| Tool | Purpose |
|---|---|
| `Python 3.x` | Core language |
| `Pandas` | Data loading, cleaning, aggregation |
| `NumPy` | Numerical computation, correlation analysis |
| `Chart.js` | Interactive dashboard visualisations |
| `python-docx` | Word report generation |

---

## 📝 Data Cleaning Notes

The cleaning process resolved issues across 5 categories:

- **Structural** — duplicate rows, whitespace in string columns
- **Encoding** — mixed Attrition values (`yes/no/1/0/TRUE/FALSE`), Department/Gender casing
- **Outliers** — impossible age values (`-5`, `999`), negative salaries, out-of-range satisfaction scores
- **Logic errors** — tenure fields that violated cross-column constraints
- **Type corrections** — float columns cast to int, HireDate converted to datetime

See `DataCleaning_AuditReport.docx` for the full 24-step cleaning log with before/after counts for every issue.

---

## 📌 Key Takeaway

> Attrition at 34.9% is not random. It concentrates in employees who work overtime, have poor work-life balance, travel frequently, or have been in the same role for 10+ years. Targeted interventions in these segments — rather than broad culture programmes — are most likely to reduce turnover efficiently.

---

*Dataset: employee_attrition_final.csv — 47,566 records, 46 columns, 100% complete after cleaning*  
*Analysis: Data Analytics Team | April 2026*
