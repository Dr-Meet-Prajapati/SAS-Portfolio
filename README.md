# 👨‍💻 Dr. Meet Prajapati — SAS Programming Portfolio

<div align="center">

![SAS](https://img.shields.io/badge/SAS-Base%20SAS-1158D4?style=for-the-badge&logo=sas&logoColor=white)
![SQL](https://img.shields.io/badge/PROC%20SQL-Clinical%20Queries-BC8CFF?style=for-the-badge)
![Healthcare](https://img.shields.io/badge/Domain-Healthcare%20Analytics-3FB950?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Available%20for%20Hire-E3B341?style=for-the-badge)

**Interactive GitHub-style SAS Programming Portfolio built on a real 500-patient healthcare dataset.**

[🌐 Live Portfolio](https://meetprajapati.github.io) • [📊 Analytics Dashboard](#-analytics-dashboard) • [💻 Code Snippets](#-sas-code-coverage) • [📁 Projects](#-featured-projects)

</div>

---

## 🗂️ About This Portfolio

This portfolio demonstrates hands-on SAS programming skills applied to a **synthetic healthcare dataset** containing 500 patient records across 13 clinical variables. It covers the full data analysis pipeline — from raw CSV import to formatted reports — using industry-standard SAS procedures.

> 💡 Built as a **fresher SAS programmer** with a medical background (MBBS), combining clinical domain knowledge with data programming skills.

---

## 📊 Dataset Overview

**File:** `synthetic_healthcare_data.csv` | **Observations:** 500 | **Variables:** 13

| Variable | Type | Description |
|---|---|---|
| `Patient_ID` | Char | Unique patient identifier (P001–P500) |
| `Age` | Num | Patient age in years (18–90) |
| `Gender` | Char | Female / Male |
| `BMI` | Num | Body Mass Index |
| `Blood_Pressure` | Char | Systolic / Diastolic (mmHg) |
| `Cholesterol_Level` | Num | Total cholesterol (mg/dL) |
| `Smoker` | Char | Yes / No |
| `Diabetic` | Char | Yes / No |
| `Diagnosis` | Char | Primary diagnosis (5 categories) |
| `Treatment_Cost` | Num | Cost in USD ($106 – $4,994) |
| `Admission_Date` | Date | Admit date (YYYY-MM-DD) |
| `Discharge_Date` | Date | Discharge date |
| `Outcome` | Char | Recovered / Referred / Deceased |

### 📈 Key Statistics (via PROC MEANS)

```
N = 500     Mean Age = 54.1     Mean BMI = 28.8
Avg Cost = $2,514     Avg LOS = 7.3 days
Diabetic Rate = 52.2%     Smoker Rate = 47.4%
```

---

## 💻 SAS Code Coverage

### 1️⃣ PROC IMPORT — Data Ingestion

```sas
/* Import synthetic healthcare CSV */
PROC IMPORT
    DATAFILE = '/data/synthetic_healthcare_data.csv'
    OUT      = work.health_data
    DBMS     = CSV
    REPLACE;
    GETNAMES = YES;
RUN;
```

---

### 2️⃣ DATA Step — Transformations

```sas
DATA work.clean_patients;
    SET work.health_data;

    /* Derived variables */
    LOS_Days = Discharge_Date - Admit_Date;

    /* BMI categorisation */
    IF      BMI < 18.5 THEN BMI_Cat = 'Underweight';
    ELSE IF BMI < 25.0 THEN BMI_Cat = 'Normal';
    ELSE IF BMI < 30.0 THEN BMI_Cat = 'Overweight';
    ELSE                    BMI_Cat = 'Obese';

    KEEP Patient_ID Age Gender BMI BMI_Cat Diagnosis
         Treatment_Cost LOS_Days Outcome;
RUN;
```

---

### 3️⃣ PROC SQL — Clinical Aggregations

```sas
/* Average cost and patient count by diagnosis */
PROC SQL;
    SELECT
        Diagnosis,
        COUNT(*)                      AS N,
        ROUND(AVG(Treatment_Cost), 2) AS Avg_Cost,
        MAX(Treatment_Cost)           AS Max_Cost
    FROM  work.clean_patients
    GROUP BY Diagnosis
    HAVING COUNT(*) > 5
    ORDER BY Avg_Cost DESC;
QUIT;
```

**Output — Avg Treatment Cost by Diagnosis:**

| Diagnosis | N | Avg Cost |
|---|---|---|
| Heart Disease | 90 | $2,665 |
| Hypertension | 106 | $2,600 |
| Flu | 85 | $2,544 |
| Pneumonia | 104 | $2,400 |
| Diabetes | 115 | $2,398 |

---

### 4️⃣ SAS Macros — Reusable Code

```sas
/* Parameterized summary macro */
%MACRO summarize(dsn=, var=, grp=);
    PROC MEANS DATA=&dsn N MEAN STD MIN MAX;
        CLASS &grp;
        VAR   &var;
        TITLE "Summary of &var by &grp";
    RUN;
%MEND summarize;

/* Call for multiple variables */
%summarize(dsn=work.clean_patients, var=Age,           grp=Diagnosis);
%summarize(dsn=work.clean_patients, var=BMI,           grp=Gender);
%summarize(dsn=work.clean_patients, var=Treatment_Cost, grp=Diagnosis);
```

---

### 5️⃣ PROC MEANS & PROC FREQ

```sas
/* Summary statistics by Diagnosis */
PROC MEANS DATA=work.clean_patients N MEAN STD MIN MAX MEDIAN;
    VAR   Age BMI Treatment_Cost LOS_Days;
    CLASS Diagnosis;
RUN;

/* Cross-tabulation with Chi-Square */
PROC FREQ DATA=work.clean_patients;
    TABLES Gender * Diagnosis / CHISQ ROW COL;
    TITLE 'Gender × Diagnosis Cross-Tabulation';
RUN;
```

---

### 6️⃣ PROC SORT — Deduplication

```sas
/* Remove duplicate Patient_IDs */
PROC SORT DATA=work.clean_patients
           OUT   =work.unique
           DUPOUT=work.removed_dups
           NODUPKEY;
    BY Patient_ID;
RUN;
```

---

### 7️⃣ ODS — Professional Reports

```sas
/* Export clinical report to PDF */
ODS PDF FILE='/output/clinical_report.pdf'
    STYLE=Journal STARTPAGE=NO;

    PROC REPORT DATA=work.clean_patients NOWD;
        COLUMN Diagnosis Patient_ID Age Treatment_Cost Outcome;
        DEFINE Diagnosis      / GROUP  'Diagnosis';
        DEFINE Age            / ANALYSIS MEAN FORMAT=5.1 'Mean Age';
        DEFINE Treatment_Cost / ANALYSIS MEAN FORMAT=DOLLAR10.2 'Avg Cost';
        BREAK AFTER Diagnosis / SUMMARIZE SUPPRESS;
        RBREAK AFTER          / SUMMARIZE;
        TITLE    'Department-wise Clinical Summary';
        FOOTNOTE "Generated: &SYSDATE";
    RUN;

ODS PDF CLOSE;
```

---

### 8️⃣ ARRAY & FIRST./LAST.

```sas
/* Array — replace missing vitals with 0 */
DATA work.vitals_clean;
    SET work.clean_patients;
    ARRAY vitals{4} Systolic Diastolic Pulse Temp;
    DO i = 1 TO 4;
        IF vitals{i} = . THEN vitals{i} = 0;
    END;
    DROP i;
RUN;

/* FIRST./LAST. — first admission per patient */
DATA work.first_visit;
    SET work.sorted_patients;
    BY Patient_ID;
    IF FIRST.Patient_ID;
RUN;
```

---

## 📁 Featured Projects

| Repository | Description | Key PROCs |
|---|---|---|
| 📊 `healthcare-analytics-sas` | End-to-end pipeline on 500-patient dataset | IMPORT, DATA Step, SQL, MEANS, ODS |
| 🔄 `sas-macro-library` | Production-ready reusable macro collection | %MACRO, %DO, %IF, %SYSFUNC |
| 🗃️ `proc-sql-clinical-queries` | JOIN, GROUP BY, subquery patterns | PROC SQL, CREATE TABLE |
| 📋 `ods-reporting-templates` | Clinical PDF/Excel/HTML report templates | ODS, PROC REPORT |
| 🧮 `data-step-transformations` | BMI, LOS, ARRAY, LAG, FIRST./LAST. | DATA Step, PROC FORMAT |
| 🔬 `proc-statistical-analysis` | Normality test, Pearson correlation | PROC UNIVARIATE, PROC CORR |
| 🔀 `proc-sort-deduplication` | Multi-key sort, NODUPKEY, DUPOUT | PROC SORT |
| 📐 `proc-transpose-reshape` | Wide↔Long reshape patterns | PROC TRANSPOSE |

---

## 🏆 Skills

```
SAS Base Programming     ████████████████████  92%
PROC SQL                 ██████████████████░░  88%
PROC MEANS / FREQ        █████████████████░░░  85%
SAS Macros               ████████████████░░░░  80%
ODS Output               ███████████████░░░░░  78%
PROC REPORT              ███████████████░░░░░  75%
DATA Step (Advanced)     ████████████████████  92%
Healthcare Domain        ████████████████████  95%
```

---

## 🚀 Quick Reference Cheat Sheet

| PROC / Concept | Primary Use |
|---|---|
| `PROC IMPORT` | Read CSV / Excel / TAB files into SAS |
| `DATA Step` | Row-by-row manipulation, new variables, logic |
| `PROC SORT` | Sort data, remove duplicates with `NODUPKEY` |
| `PROC SQL` | SELECT, JOIN, GROUP BY, subqueries |
| `%MACRO / %LET` | Reusable parameterized code blocks |
| `PROC MEANS` | N, MEAN, STD, MIN, MAX by CLASS group |
| `PROC FREQ` | Frequency counts, cross-tabs, chi-square |
| `DATA MERGE` | Horizontal join by key variable |
| `PROC APPEND` | Stack datasets vertically |
| `ARRAY` | Process multiple variables in a loop |
| `PROC FORMAT` | Custom display formats for codes/ranges |
| `PROC REPORT` | Tabular reports with BREAK / RBREAK |
| `ODS EXCEL/PDF/HTML` | Export output to Excel, PDF, HTML |
| `PROC TRANSPOSE` | Reshape wide ↔ long |
| `FIRST. / LAST.` | Detect first/last row in a BY group |
| `LAG()` | Access previous row value in DATA Step |

---

## 📂 Repository Structure

```
📦 healthcare-analytics-sas
 ┣ 📂 data
 ┃ ┣ 📄 synthetic_healthcare_data.csv
 ┃ ┗ 📄 healthportfolio.sas7bdat
 ┣ 📂 programs
 ┃ ┣ 📄 01_import_data.sas
 ┃ ┣ 📄 02_data_step_clean.sas
 ┃ ┣ 📄 03_proc_sql_analysis.sas
 ┃ ┣ 📄 04_macros.sas
 ┃ ┣ 📄 05_proc_means_freq.sas
 ┃ ┗ 📄 06_ods_reports.sas
 ┣ 📂 output
 ┃ ┣ 📄 clinical_report.pdf
 ┃ ┗ 📄 patient_report.xlsx
 ┣ 📄 index.html          ← Interactive Portfolio Dashboard
 ┗ 📄 README.md
```

---

## 📬 Contact

<div align="center">

| Platform | Link |
|---|---|
| 🌐 Portfolio | [meetprajapati.github.io](https://meetprajapati.github.io) |
| 💼 LinkedIn | [linkedin.com/in/meetprajapati](https://linkedin.com/in/meetprajapati) |
| 📧 Email | meet.prajapati@email.com |

**Open to Clinical SAS Programmer · Biostatistics Analyst · Healthcare Data Analyst roles**

</div>

---

<div align="center">

*© Dr. Meet Prajapati — SAS Programming Portfolio*

![Visitors](https://visitor-badge.laobi.icu/badge?page_id=meetprajapati.healthcare-analytics-sas)

</div>
