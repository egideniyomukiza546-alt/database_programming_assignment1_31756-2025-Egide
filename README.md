# Database Programming Assignment I: CTEs & SQL Window Functions

## Course Information
- **Course Title:** C11665 - DPR400210: Database Programming  
- **Instructor:** Eric Maniraguha  
- **Student Name:** NIYOMUKIZA Egide  
- **Student ID:** 31756/2025  
- **Group:** One  
- **Assignment Date:**  June 2026  

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

 
- **CREATION OF OF  TABLES** Queries** (Simple, Multiple, Recursive, Aggregation, Joins
- **CTE Queries** (Simple, Multiple, Recursive, Aggregation, Joins)  
- **Window Functions** (Ranking, Aggregate, Navigation, Distribution)  
- **Analysis Queries** (Descriptive, Monthly Trends, Diagnostic, Prescriptive)

```sql


SET ECHO ON
SET LINESIZE 200
SET PAGESIZE 100

CREATE TABLE Doctors (
    doctor_id INT PRIMARY KEY,
    first_name VARCHAR2(50),
    last_name VARCHAR2(50),
    specialty VARCHAR2(100),
    hire_date DATE,
    salary NUMBER(10,2),
    department VARCHAR2(50)
);
COMMIT;

CREATE TABLE Patients (
    patient_id INT PRIMARY KEY,
    first_name VARCHAR2(50),
    last_name VARCHAR2(50),
    date_of_birth DATE,
    gender VARCHAR2(10),
    phone VARCHAR2(15),
    email VARCHAR2(100),
    address VARCHAR2(200)
);
COMMIT;

CREATE TABLE Admissions (
    admission_id INT PRIMARY KEY,
    patient_id INT,
    doctor_id INT,
    admission_date DATE,
    discharge_date DATE,
    diagnosis VARCHAR2(200),
    room_number VARCHAR2(10),
    total_cost NUMBER(10,2),
    FOREIGN KEY (patient_id) REFERENCES Patients(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES Doctors(doctor_id)
);
COMMIT;


-- Doctors Data
INSERT INTO Doctors VALUES (1, 'John', 'Smith', 'Cardiology', DATE '2015-06-01', 250000.00, 'Cardiology');
INSERT INTO Doctors VALUES (2, 'Sarah', 'Johnson', 'Neurology', DATE '2016-08-15', 230000.00, 'Neurology');
INSERT INTO Doctors VALUES (3, 'Michael', 'Williams', 'Orthopedics', DATE '2018-01-10', 210000.00, 'Orthopedics');
INSERT INTO Doctors VALUES (4, 'Emily', 'Brown', 'Pediatrics', DATE '2017-03-20', 190000.00, 'Pediatrics');
INSERT INTO Doctors VALUES (5, 'David', 'Jones', 'Cardiology', DATE '2019-05-05', 220000.00, 'Cardiology');
INSERT INTO Doctors VALUES (6, 'Lisa', 'Garcia', 'Neurology', DATE '2020-07-12', 200000.00, 'Neurology');
INSERT INTO Doctors VALUES (7, 'Robert', 'Miller', 'Orthopedics', DATE '2018-09-01', 215000.00, 'Orthopedics');
INSERT INTO Doctors VALUES (8, 'Jennifer', 'Davis', 'Pediatrics', DATE '2016-11-25', 195000.00, 'Pediatrics');
INSERT INTO Doctors VALUES (9, 'William', 'Rodriguez', 'Cardiology', DATE '2017-02-14', 240000.00, 'Cardiology');
INSERT INTO Doctors VALUES (10, 'Patricia', 'Martinez', 'Neurology', DATE '2019-08-30', 225000.00, 'Neurology');

-- Patients Data
INSERT INTO Patients VALUES (101, 'James', 'Ndayisaba', DATE '1985-03-15', 'Male', '555-0101', 'james.a@email.com', '123 Oak St, City');
INSERT INTO Patients VALUES (102, 'Mary', 'Uwimana', DATE '1990-07-22', 'Female', '555-0102', 'mary.t@email.com', '456 Pine Ave, City');
INSERT INTO Patients VALUES (103, 'Robert', 'Habimana', DATE '1978-11-08', 'Male', '555-0103', 'robert.c@email.com', '789 Elm Rd, City');
INSERT INTO Patients VALUES (104, 'Patricia', 'Mukamana', DATE '1995-02-28', 'Female', '555-0104', 'patricia.w@email.com', '321 Maple Dr, City');
INSERT INTO Patients VALUES (105, 'Michael', 'Bizimana', DATE '1982-09-12', 'Male', '555-0105', 'michael.h@email.com', '654 Birch Ln, City');
INSERT INTO Patients VALUES (106, 'Jennifer', 'Nshimiyimana', DATE '2000-06-05', 'Female', '555-0106', 'jennifer.m@email.com', '987 Cedar Ct, City');
INSERT INTO Patients VALUES (107, 'William', 'Nsengiyumva', DATE '1975-04-18', 'Male', '555-0107', 'william.j@email.com', '147 Willow Way, City');
INSERT INTO Patients VALUES (108, 'Linda', 'Imanishimwe', DATE '1988-12-30', 'Female', '555-0108', 'linda.t@email.com', '258 Spruce St, City');
INSERT INTO Patients VALUES (109, 'David', 'Manzi', DATE '1992-08-14', 'Male', '555-0109', 'david.g@email.com', '369 Ash Ave, City');
INSERT INTO Patients VALUES (110, 'Sarah', 'Niyonkuru', DATE '1980-01-25', 'Female', '555-0110', 'sarah.m@email.com', '741 Poplar Dr, City');

-- Admissions Data
INSERT INTO Admissions VALUES (1001,105,5,DATE '2026-01-05',DATE '2026-01-08','Chest Pain','301B',6200.00);
INSERT INTO Admissions VALUES (1002,102,2,DATE '2026-01-10',DATE '2026-01-15','Migraine','302B',3200.00);
INSERT INTO Admissions VALUES (1003,103,7,DATE '2026-01-12',DATE '2026-01-18','Knee Surgery','303C',7800.00);
INSERT INTO Admissions VALUES (1004,110,4,DATE '2026-01-20',DATE '2026-01-22','Fever','304A',1500.00);
INSERT INTO Admissions VALUES (1005,109,9,DATE '2026-02-01',DATE '2026-02-05','Heart Arrhythmia','301C',6200.00);
INSERT INTO Admissions VALUES (1006,106,6,DATE '2026-02-03',DATE '2026-02-06','Seizure','302C',5400.00);
INSERT INTO Admissions VALUES (1007,107,3,DATE '2026-02-10',DATE '2026-02-20','Fracture','303A',9500.00);
INSERT INTO Admissions VALUES (1008,108,8,DATE '2026-02-15',DATE '2026-02-17','Childhood Illness','304B',2100.00);
INSERT INTO Admissions VALUES (1009,101,1,DATE '2026-03-01',DATE '2026-03-05','Heart Valve Issue','301A',8800.00);
INSERT INTO Admissions VALUES (1010,110,10,DATE '2026-03-03',DATE '2026-03-06','Head Injury','302A',4600.00);
INSERT INTO Admissions VALUES (1011,104,4,DATE '2026-03-08',DATE '2026-03-09','Allergic Reaction','304A',800.00);
INSERT INTO Admissions VALUES (1012,102,6,DATE '2026-03-12',DATE '2026-03-18','Stroke','302B',9400.00);
INSERT INTO Admissions VALUES (1013,103,7,DATE '2026-03-15',DATE '2026-03-22','Hip Replacement','303C',12000.00);
INSERT INTO Admissions VALUES (1014,105,5,DATE '2026-04-01',DATE '2026-04-05','Chest Pain','301B',4800.00);
INSERT INTO Admissions VALUES (1015,106,10,DATE '2026-04-03',DATE '2026-04-08','Brain Tumor','302C',15000.00);
INSERT INTO Admissions VALUES (1016,107,3,DATE '2026-04-10',DATE '2026-04-18','Spinal Surgery','303A',13500.00);
INSERT INTO Admissions VALUES (1017,108,8,DATE '2026-04-15',DATE '2026-04-16','Ear Infection','304B',1200.00);
INSERT INTO Admissions VALUES (1018,109,9,DATE '2026-04-20',DATE '2026-04-28','Heart Bypass','301C',22000.00);
INSERT INTO Admissions VALUES (1019,101,1,DATE '2026-05-05',DATE '2026-05-10','Angina','301A',5200.00);
INSERT INTO Admissions VALUES (1020,102,2,DATE '2026-05-12',DATE '2026-05-15','Concussion','302B',3800.00);

COMMIT;

CREATE TABLE Departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR2(100),
    parent_dept_id INT,
    FOREIGN KEY (parent_dept_id) REFERENCES Departments(dept_id)
);
COMMIT;

INSERT INTO Departments VALUES (1, 'Healthcare', NULL);
INSERT INTO Departments VALUES (2, 'Medical', 1);
INSERT INTO Departments VALUES (3, 'Surgical', 1);
INSERT INTO Departments VALUES (4, 'Cardiology', 2);
INSERT INTO Departments VALUES (5, 'Neurology', 2);
INSERT INTO Departments VALUES (6, 'Orthopedics', 3);
INSERT INTO Departments VALUES (7, 'Pediatrics', 3);

COMMIT;

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
Diagnostic Analysis.png
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

