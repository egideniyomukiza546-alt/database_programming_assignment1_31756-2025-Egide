# Database Programming Assignment I: CTEs & SQL Window Functions

## Course Information
- **Course Title:** C11665 - DPR400210: Database Programming  
- **Instructor:** Eric Maniraguha  
- **Student Name:** NIYOMUKIZA Egide  
- **Student ID:** 31756/2025  
- **Group:** One  
- **Assignment Date:** 29 June 2026  

---

## Business Scenario
MediCare Hospital needs a comprehensive database system to manage patient admissions, doctor assignments, and treatment costs. The system aims to analyze doctor performance, patient visit patterns, and revenue trends to optimize healthcare delivery and financial management.

---
## Business Problem

MediCare Hospital, a mid-sized healthcare facility, faces challenges in managing and analyzing patient admission data, doctor performance, and treatment costs. The hospital currently lacks an integrated system that provides actionable insights from its operational data. 

**Key Challenges:**
- Difficulty identifying high-cost admissions and their root causes
- Inability to effectively measure and compare doctor performance
- Lack of visibility into patient visit patterns and readmission rates
- No systematic method for analyzing revenue trends across departments
- Limited ability to make data-driven decisions for cost optimization

**Solution:**
This database system is designed to address these challenges by:
1. Tracking patient admissions, doctor assignments, and treatment costs
2. Providing analytical tools for performance measurement
3. Enabling identification of cost-saving opportunities
4. Supporting strategic decision-making through data-driven insights

The system aims to optimize healthcare delivery, improve patient outcomes, and enhance financial management through comprehensive data analysis using Common Table Expressions (CTEs) and Window Functions.
## Database Schema

### Tables

1. **Doctors** – stores physician information  
   - `doctor_id` (PK), `first_name`, `last_name`, `specialty`, `hire_date`, `salary`, `department`

2. **Patients** – stores patient information  
   - `patient_id` (PK), `first_name`, `last_name`, `date_of_birth`, `gender`, `phone`, `email`

3. **Admissions** – stores admission records  
   - `admission_id` (PK), `patient_id` (FK → Patients), `doctor_id` (FK → Doctors)  
   - `admission_date`, `discharge_date`, `diagnosis`, `room`, `total_cost`

### Relationships
- Doctors **1---*** Admissions (one doctor handles many admissions)  
- Patients **1---*** Admissions (one patient can have multiple admissions)

### ER Diagram
![ER Diagram](hospital_management_erd.png)

---

## SQL Script – Full Implementation

The entire SQL script is provided below. It includes:

- **CTE Queries** (Simple, Multiple, Recursive, Aggregation, Joins)  
- **Window Functions** (Ranking, Aggregate, Navigation, Distribution)  
- **Analysis Queries** (Descriptive, Monthly Trends, Diagnostic, Prescriptive)

Before running the script, ensure that the tables `Admissions`, `Patients`, and `Doctors` exist and contain data. The script itself creates the `Departments` table on the fly for the recursive CTE example.

```sql


SET ECHO ON
SET LINESIZE 200
SET PAGESIZE 100

-- CTE QUERIES


-- CTE 1: Simple CTE - High-Cost Admissions
WITH HighCostAdmissions AS (
    SELECT 
        admission_id,
        patient_id,
        doctor_id,
        total_cost,
        diagnosis
    FROM Admissions
    WHERE total_cost > 5000.00
)
SELECT 
    h.admission_id,
    p.first_name || ' ' || p.last_name AS patient_name,
    h.diagnosis,
    h.total_cost
FROM HighCostAdmissions h
JOIN Patients p ON h.patient_id = p.patient_id
ORDER BY h.total_cost DESC;

-- CTE 2: Multiple CTEs - Doctor Performance Analysis
WITH DoctorAdmissionCount AS (
    SELECT 
        doctor_id,
        COUNT(*) AS total_admissions
    FROM Admissions
    GROUP BY doctor_id
),
DoctorRevenue AS (
    SELECT 
        doctor_id,
        SUM(total_cost) AS total_revenue
    FROM Admissions
    GROUP BY doctor_id
)
SELECT 
    d.doctor_id,
    d.first_name || ' ' || d.last_name AS doctor_name,
    d.specialty,
    NVL(dac.total_admissions, 0) AS admissions_count,
    NVL(dr.total_revenue, 0) AS revenue_generated
FROM Doctors d
LEFT JOIN DoctorAdmissionCount dac ON d.doctor_id = dac.doctor_id
LEFT JOIN DoctorRevenue dr ON d.doctor_id = dr.doctor_id
ORDER BY revenue_generated DESC;

-- CTE 3: Recursive CTE - Department Hierarchy

CREATE TABLE Departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR2(100),
    parent_dept_id INT,
    FOREIGN KEY (parent_dept_id) REFERENCES Departments(dept_id)
);

INSERT INTO Departments VALUES (1, 'Healthcare', NULL);
INSERT INTO Departments VALUES (2, 'Medical', 1);
INSERT INTO Departments VALUES (3, 'Surgical', 1);
INSERT INTO Departments VALUES (4, 'Cardiology', 2);
INSERT INTO Departments VALUES (5, 'Neurology', 2);
INSERT INTO Departments VALUES (6, 'Orthopedics', 3);
INSERT INTO Departments VALUES (7, 'Pediatrics', 3);

WITH DeptHierarchy (dept_id, dept_name, parent_dept_id, tree_level, path_hierarchy) AS (
    SELECT 
        dept_id,
        dept_name,
        parent_dept_id,
        0 AS tree_level,
        CAST(dept_name AS VARCHAR2(500)) AS path_hierarchy
    FROM Departments
    WHERE parent_dept_id IS NULL
    
    UNION ALL
    
    SELECT 
        d.dept_id,
        d.dept_name,
        d.parent_dept_id,
        dh.tree_level + 1,
        CAST(dh.path_hierarchy || ' -> ' || d.dept_name AS VARCHAR2(500))
    FROM Departments d
    INNER JOIN DeptHierarchy dh ON d.parent_dept_id = dh.dept_id
)
SELECT 
    tree_level,
    LPAD(' ', tree_level * 2) || dept_name AS department_tree,
    path_hierarchy
FROM DeptHierarchy
ORDER BY tree_level, dept_name;

-- CTE 4: CTE with Aggregation - Monthly Revenue Analysis
WITH MonthlyRevenue AS (
    SELECT 
        TO_CHAR(admission_date, 'YYYY-MM') AS month_year,
        COUNT(*) AS total_admissions,
        SUM(total_cost) AS total_revenue,
        AVG(total_cost) AS avg_cost_per_admission
    FROM Admissions
    GROUP BY TO_CHAR(admission_date, 'YYYY-MM')
),
RevenueRank AS (
    SELECT 
        month_year,
        total_revenue,
        total_admissions,
        avg_cost_per_admission,
        RANK() OVER (ORDER BY total_revenue DESC) AS revenue_rank
    FROM MonthlyRevenue
)
SELECT 
    month_year,
    total_revenue,
    total_admissions,
    avg_cost_per_admission,
    revenue_rank
FROM RevenueRank
ORDER BY revenue_rank;

-- CTE 5: CTE with JOIN - Patient Visit History
WITH PatientVisits AS (
    SELECT 
        p.patient_id,
        p.first_name || ' ' || p.last_name AS patient_name,
        a.admission_date,
        a.discharge_date,
        a.diagnosis,
        a.total_cost,
        (a.discharge_date - a.admission_date) AS length_of_stay,
        d.first_name || ' ' || d.last_name AS doctor_name,
        d.specialty
    FROM Admissions a
    JOIN Patients p ON a.patient_id = p.patient_id
    JOIN Doctors d ON a.doctor_id = d.doctor_id
)
SELECT 
    patient_name,
    COUNT(*) AS visit_count,
    SUM(total_cost) AS total_medical_bill,
    AVG(length_of_stay) AS avg_stay_days,
    MAX(total_cost) AS most_expensive_visit
FROM PatientVisits
GROUP BY patient_name
HAVING COUNT(*) > 1
ORDER BY total_medical_bill DESC;

-- WINDOW FUNCTION QUERIES


-- Ranking Functions
-- ROW_NUMBER()
WITH RankedAdmissions AS (
    SELECT 
        a.admission_id,
        p.first_name || ' ' || p.last_name AS patient_name,
        d.first_name || ' ' || d.last_name AS doctor_name,
        a.total_cost,
        ROW_NUMBER() OVER (
            PARTITION BY a.doctor_id 
            ORDER BY a.total_cost DESC
        ) AS cost_rank
    FROM Admissions a
    JOIN Patients p ON a.patient_id = p.patient_id
    JOIN Doctors d ON a.doctor_id = d.doctor_id
)
SELECT 
    doctor_name,
    patient_name,
    total_cost,
    cost_rank
FROM RankedAdmissions
WHERE cost_rank <= 3
ORDER BY doctor_name, cost_rank;

-- RANK() and DENSE_RANK()
WITH DepartmentRevenue AS (
    SELECT 
        d.department,
        SUM(a.total_cost) AS total_revenue,
        COUNT(a.admission_id) AS admission_count
    FROM Doctors d
    JOIN Admissions a ON d.doctor_id = a.doctor_id
    GROUP BY d.department
)
SELECT 
    department,
    total_revenue,
    admission_count,
    RANK() OVER (ORDER BY total_revenue DESC) AS revenue_rank,
    DENSE_RANK() OVER (ORDER BY total_revenue DESC) AS dense_revenue_rank
FROM DepartmentRevenue
ORDER BY revenue_rank;

-- PERCENT_RANK()
WITH CostPercentile AS (
    SELECT 
        a.admission_id,
        p.first_name || ' ' || p.last_name AS patient_name,
        a.total_cost,
        PERCENT_RANK() OVER (ORDER BY a.total_cost) AS cost_percentile,
        NTILE(4) OVER (ORDER BY a.total_cost) AS cost_quartile
    FROM Admissions a
    JOIN Patients p ON a.patient_id = p.patient_id
)
SELECT 
    patient_name,
    total_cost,
    ROUND(cost_percentile * 100, 2) AS percentile_rank,
    cost_quartile,
    CASE 
        WHEN cost_quartile = 1 THEN 'Lowest 25%'
        WHEN cost_quartile = 2 THEN 'Lower Middle 25%'
        WHEN cost_quartile = 3 THEN 'Upper Middle 25%'
        WHEN cost_quartile = 4 THEN 'Highest 25%'
    END AS cost_category
FROM CostPercentile
ORDER BY total_cost DESC;

-- Aggregate Window Functions
-- SUM() OVER()
SELECT 
    a.admission_date,
    p.first_name || ' ' || p.last_name AS patient_name,
    a.total_cost,
    SUM(a.total_cost) OVER (ORDER BY a.admission_date) AS cumulative_revenue,
    SUM(a.total_cost) OVER (PARTITION BY EXTRACT(MONTH FROM a.admission_date) 
                              ORDER BY a.admission_date) AS monthly_cumulative
FROM Admissions a
JOIN Patients p ON a.patient_id = p.patient_id
ORDER BY a.admission_date;

-- AVG() OVER()
SELECT 
    a.admission_date,
    a.total_cost,
    AVG(a.total_cost) OVER (ORDER BY a.admission_date 
                            ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING) AS moving_avg_5days,
    AVG(a.total_cost) OVER (PARTITION BY EXTRACT(MONTH FROM a.admission_date)) AS monthly_avg
FROM Admissions a
ORDER BY a.admission_date;

-- MIN/MAX OVER()
WITH DepartmentStats AS (
    SELECT 
        a.admission_id,
        d.department,
        a.total_cost,
        MIN(a.total_cost) OVER (PARTITION BY d.department) AS dept_min,
        MAX(a.total_cost) OVER (PARTITION BY d.department) AS dept_max,
        AVG(a.total_cost) OVER (PARTITION BY d.department) AS dept_avg
    FROM Admissions a
    JOIN Doctors d ON a.doctor_id = d.doctor_id
)
SELECT 
    department,
    total_cost,
    dept_min,
    dept_max,
    dept_avg,
    CASE 
        WHEN total_cost = dept_min THEN 'Lowest in Department'
        WHEN total_cost = dept_max THEN 'Highest in Department'
        WHEN total_cost > dept_avg THEN 'Above Department Average'
        WHEN total_cost < dept_avg THEN 'Below Department Average'
    END AS cost_comparison
FROM DepartmentStats
ORDER BY department, total_cost DESC;

-- Navigation Functions
-- LAG()
SELECT 
    a.admission_id,
    p.first_name || ' ' || p.last_name AS patient_name,
    a.admission_date,
    a.total_cost,
    LAG(a.total_cost, 1) OVER (PARTITION BY a.patient_id 
                                ORDER BY a.admission_date) AS previous_cost,
    LAG(a.diagnosis, 1) OVER (PARTITION BY a.patient_id 
                               ORDER BY a.admission_date) AS previous_diagnosis,
    CASE 
        WHEN a.total_cost > LAG(a.total_cost, 1) OVER (PARTITION BY a.patient_id 
                                                         ORDER BY a.admission_date) 
        THEN 'Cost Increased'
        WHEN a.total_cost < LAG(a.total_cost, 1) OVER (PARTITION BY a.patient_id 
                                                         ORDER BY a.admission_date) 
        THEN 'Cost Decreased'
        ELSE 'First Visit'
    END AS cost_trend
FROM Admissions a
JOIN Patients p ON a.patient_id = p.patient_id
ORDER BY p.patient_id, a.admission_date;

-- LEAD()
SELECT 
    p.first_name || ' ' || p.last_name AS patient_name,
    a.admission_date,
    a.diagnosis AS current_diagnosis,
    a.total_cost AS current_cost,
    LEAD(a.diagnosis, 1) OVER (PARTITION BY a.patient_id 
                                ORDER BY a.admission_date) AS next_diagnosis,
    LEAD(a.total_cost, 1) OVER (PARTITION BY a.patient_id 
                                 ORDER BY a.admission_date) AS next_cost,
    LEAD(a.admission_date, 1) OVER (PARTITION BY a.patient_id 
                                     ORDER BY a.admission_date) AS next_admission_date
FROM Admissions a
JOIN Patients p ON a.patient_id = p.patient_id
ORDER BY p.patient_id, a.admission_date;

-- Distribution Functions
-- NTILE()
SELECT 
    p.first_name || ' ' || p.last_name AS patient_name,
    a.diagnosis,
    a.total_cost,
    NTILE(4) OVER (ORDER BY a.total_cost) AS cost_quartile,
    NTILE(10) OVER (ORDER BY a.total_cost) AS cost_decile,
    CASE 
        WHEN NTILE(4) OVER (ORDER BY a.total_cost) = 1 THEN 'Low Cost'
        WHEN NTILE(4) OVER (ORDER BY a.total_cost) = 2 THEN 'Medium Low'
        WHEN NTILE(4) OVER (ORDER BY a.total_cost) = 3 THEN 'Medium High'
        WHEN NTILE(4) OVER (ORDER BY a.total_cost) = 4 THEN 'High Cost'
    END AS cost_category
FROM Admissions a
JOIN Patients p ON a.patient_id = p.patient_id
ORDER BY a.total_cost DESC;

-- CUME_DIST()
SELECT 
    a.total_cost,
    CUME_DIST() OVER (ORDER BY a.total_cost) AS cumulative_distribution,
    COUNT(*) OVER (PARTITION BY a.total_cost) AS frequency_same_cost,
    ROUND(CUME_DIST() OVER (ORDER BY a.total_cost) * 100, 2) AS percentage_below
FROM Admissions a
ORDER BY a.total_cost;

-- ANALYSIS QUERIES


-- Descriptive Analysis
SELECT 
    COUNT(*) AS total_admissions,
    SUM(total_cost) AS total_revenue,
    AVG(total_cost) AS average_cost,
    MIN(total_cost) AS cheapest_admission,
    MAX(total_cost) AS most_expensive,
    AVG(discharge_date - admission_date) AS avg_length_of_stay
FROM Admissions;

-- Monthly Trends
SELECT 
    TO_CHAR(admission_date, 'YYYY-MM') AS month,
    COUNT(*) AS admissions,
    SUM(total_cost) AS revenue,
    AVG(total_cost) AS avg_cost,
    COUNT(DISTINCT patient_id) AS unique_patients
FROM Admissions
GROUP BY TO_CHAR(admission_date, 'YYYY-MM')
ORDER BY month;

-- Diagnostic Analysis
WITH DepartmentAnalysis AS (
    SELECT 
        d.department,
        COUNT(a.admission_id) AS admission_count,
        SUM(a.total_cost) AS total_revenue,
        AVG(a.total_cost) AS avg_cost,
        AVG(a.discharge_date - a.admission_date) AS avg_stay,
        COUNT(DISTINCT a.patient_id) AS unique_patients
    FROM Doctors d
    JOIN Admissions a ON d.doctor_id = a.doctor_id
    GROUP BY d.department
)
SELECT 
    department,
    admission_count,
    total_revenue,
    avg_cost,
    avg_stay,
    RANK() OVER (ORDER BY total_revenue DESC) AS revenue_rank,
    CASE 
        WHEN avg_cost > 8000 AND avg_stay > 5 THEN 'High Cost - Long Stay'
        WHEN avg_cost > 8000 AND avg_stay <= 5 THEN 'High Cost - Short Stay'
        WHEN avg_cost <= 8000 AND avg_stay > 5 THEN 'Low Cost - Long Stay'
        ELSE 'Efficient'
    END AS performance_category
FROM DepartmentAnalysis;

-- Prescriptive Analysis
WITH DoctorEfficiency AS (
    SELECT 
        d.doctor_id,
        d.first_name || ' ' || d.last_name AS doctor_name,
        d.specialty,
        COUNT(a.admission_id) AS patient_count,
        AVG(a.total_cost) AS avg_cost,
        AVG(a.discharge_date - a.admission_date) AS avg_stay,
        RANK() OVER (ORDER BY AVG(a.total_cost) DESC) AS cost_rank,
        RANK() OVER (ORDER BY AVG(a.discharge_date - a.admission_date)) AS efficiency_rank
    FROM Doctors d
    JOIN Admissions a ON d.doctor_id = a.doctor_id
    GROUP BY d.doctor_id, d.first_name, d.last_name, d.specialty
)
SELECT 
    doctor_name,
    specialty,
    patient_count,
    avg_cost,
    avg_stay,
    CASE 
        WHEN cost_rank <= 3 AND efficiency_rank >= 4 THEN 'High Cost - Needs Cost Optimization'
        WHEN cost_rank >= 4 AND efficiency_rank >= 4 THEN 'High Cost - Needs Efficiency Improvement'
        WHEN cost_rank >= 4 AND efficiency_rank <= 3 THEN 'Efficient and Cost-Effective - Use as Model'
        WHEN cost_rank <= 3 AND efficiency_rank <= 3 THEN 'High Cost but Efficient - Need Cost Reduction'
    END AS recommendation
FROM DoctorEfficiency
WHERE patient_count > 2
ORDER BY cost_rank;

SET ECHO OFF
```
## Analysis and Findings

### Descriptive Analysis (What Happened?)

| Metric | Value |
|--------|-------|
| Total Admissions | 20 |
| Total Revenue | $143,000.00 |
| Average Cost per Admission | $7,150.00 |
| Most Expensive Admission | $22,000.00 (Heart Bypass) |
| Cheapest Admission | $800.00 (Allergic Reaction) |
| Average Length of Stay | 4.5 days |
| Busiest Month | April 2026 (5 admissions, $56,500) |
| Top Department by Revenue | Cardiology ($53,200) |
| Patients with Multiple Visits | 9 out of 10 (90%) |
| Most Frequent Patient | Mary Uwimana (3 visits) |

**Key Findings:**
- April generated 39.5% of total revenue
- Cardiology accounts for 37.2% of total hospital revenue
- 9 out of 10 patients have multiple visits (90% readmission rate)
- Top 3 doctors generate 49.6% of total revenue

---

### Diagnostic Analysis (Why Did It Happen?)

| Finding | Root Cause |
|---------|------------|
| Cardiology highest revenue | Complex heart surgeries (Bypass: $22,000, Valve: $8,800) |
| Orthopedics longest stay (7.75 days) | Post-surgery physical therapy requirements |
| Pediatrics lowest cost | Non-surgical cases, faster recovery |
| April highest revenue | Major elective surgeries scheduled in spring |
| Mary Uwimana has 3 visits | Chronic condition (Migraine → Stroke → Concussion) |
| Dr. Rodriguez highest revenue | Treats most complex Cardiology cases |
| Cost escalation | Migraine ($3,200) → Stroke ($9,400) - 194% increase |

**Key Insights:**
- Seasonal factors affect admission patterns
- Readmission rates indicate need for preventive care
- Cardiology has highest average cost ($8,866.67)

---

### Prescriptive Analysis (What Actions Should Be Taken?)

| # | Recommendation | Priority | Timeline | Expected Impact |
|---|----------------|----------|----------|-----------------|
| 1 | **Cardiology Cost Optimization** - Review top 25% expensive admissions | HIGH | 3-6 months | Reduce costs by 10-15% |
| 2 | **Orthopedics Efficiency** - Early-mobilization protocols | HIGH | 4-6 months | Reduce stay by 22% |
| 3 | **Resource Allocation** - Extra Cardiology staff in peak months | MEDIUM | Immediate | Improve service quality |
| 4 | **Pediatrics Best Practices** - Use as cost-efficiency model | MEDIUM | 6-12 months | Reduce costs by 20% |
| 5 | **Preventive Care Program** - Follow-up for chronic conditions | HIGH | 6 months | Reduce readmissions by 20% |
| 6 | **Doctor Performance Review** - Balance cost vs. quality | MEDIUM | 4 months | Optimize outcomes |
| 7 | **Hierarchy Streamlining** - Use ER diagram structure | LOW | 3 months | Improve administration |
| 8 | **High-Cost Patient Prediction** - Early intervention system | HIGH | 6-12 months | Improve outcomes |

**Expected Annual Savings:** $20,000 - $30,000

---
## References

1. Oracle 21c SQL Language Reference
2. Course Materials - DPR400210: Database Programming
3. Oracle Live SQL Documentation
4. SQL*Plus User's Guide

   ## Academic Integrity Statement

I hereby declare that this assignment is my own original work. All sources used have been properly cited, and I have not copied or collaborated with others without permission.

**Signature:** NIYOMUKIZA Egide
**Date:** 29 June 2026

