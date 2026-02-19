# Student-Placement-Overview
This dashboard analyzes internship trends by gender using dynamic parameters. It allows users to toggle between Male, Female, and Aggregated views to compare career preparation metrics. By utilizing calculated fields and LOD expressions, the tool provides a weighted average of internship counts, offering clear insights into workforce entry data.

Key Features
Dynamic Filtering: A single parameter control to switch between specific gender demographics or a combined view.

Weighted Aggregations: Custom logic ensuring the "Both" option calculates a true average across the entire dataset rather than a simple average of averages.

Interactive Visualization: Designed for clear comparison of professional development milestones.

Original Data Source:
[Student_Placement_Dateset_2025](https://www.kaggle.com/datasets/alhamdulliah123/student-placement-and-skills-analytics-dataset-2025/data)

Data Hygiene & Pre-processing: Standardizing the 2025 Placement Dataset (Excel):

Before running the SQL analytics, the raw dataset student_placement_skills_2025 underwent a standardization process in Excel to ensure data integrity.

  1. Text Normalization Problem: Inconsistent casing.
  2. Cleaning: Applied TRIM() to remove accidental leading or trailing spaces that could break GROUP BY SQL queries.
  3. Handling Nulls & Missing Values
  4. Skills Scores: Any missing test scores were cross-referenced or marked with the median score to avoid skewing the distribution.
  5. Data Type Formatting Numeric Precision: The CGPA column was formatted to 2 decimal places. Currency: The Salary_Offered_USD column was set to "Currency" format without symbols for easy CSV export. Boolean Mapping: The Placement_Offer column was standardized to a strict "Yes/No" toggle.
  6. Used Data Validation in Excel to ensure: Technical_Skills_Score_100 remains between 0 and 100. Age falls within a realistic range (18 - 30 years).

SQL Queries:

# Inspecting the Table
SELECT *
FROM student_placement_skills_2025
LIMIT 10;

# Selecting high-performers who scored above 80 in ALL three skill categories to find high achievers
SELECT Student_Id, CGPA, Placement_Offer
FROM student_placement_skills_2025
WHERE Technical_Skills_Score_100 > 80
  AND Communication_Skills_Score_100 > 80
  AND Aptitude_Test_Score_100 > 80;

# Selecting the average salary offered by Gender and viewing major discrepancies
SELECT gender, age, ROUND(AVG(Salary_Offered_USD), 2) AS avg_salary_offered
FROM student_placement_skills_2025
GROUP BY 1, 2
ORDER BY 2, 1;

# Finding degrees with an average salary greater than 11,000
SELECT 
    Degree, 
    COUNT(Student_Id) AS Total_Students,
    AVG(Salary_Offered_USD) AS Average_Salary
FROM student_placement_skills_2025
WHERE Placement_Offer = 'Yes'  -- Only looking at students who actually got jobs
GROUP BY Degree
HAVING AVG(Salary_Offered_USD) > 11000
ORDER BY Average_Salary DESC;

# Ranking students by GPA within their respective degrees and only taking the top 10
WITH ranking_system AS (SELECT 
    Student_Id, 
    Degree, 
    CGPA,
    DENSE_RANK() OVER (PARTITION BY Degree ORDER BY CGPA DESC) AS Rank_In_Degree
FROM student_placement_skills_2025)
SELECT *
FROM ranking_system 
WHERE Rank_In_Degree <= 10;

# Comparing each student's salary to the student ranked immediately below them
WITH salary_drop AS (SELECT 
    Student_Id, 
    Salary_Offered_USD,
    LAG(Salary_Offered_USD) OVER (ORDER BY Salary_Offered_USD DESC) AS Next_Higher_Salary,
    Salary_Offered_USD - LEAD(Salary_Offered_USD) OVER (ORDER BY Salary_Offered_USD DESC) AS Salary_Drop_Off,
    ROW_NUMBER() OVER(ORDER BY Salary_Offered_USD DESC) AS rn
FROM student_placement_skills_2025
WHERE Placement_Offer = 'Yes')
SELECT Student_Id, Salary_Offered_USD, Next_Higher_Salary, ROUND(Salary_Drop_Off, 2) AS Salary_Drop_Off, 
	ROUND((Next_Higher_Salary - Salary_Offered_USD)/Next_Higher_Salary, 3) AS pct_change
FROM salary_drop;

# Analyzing the impact of 'Extra' activities on Salary
SELECT 
    Internships_Count,
    Certifications_Count,
    ROUND(AVG(Technical_Skills_Score_100), 1) AS Avg_Tech_Score,
    ROUND(AVG(Salary_Offered_USD), 2) AS Avg_Salary
FROM student_placement_skills_2025
GROUP BY Internships_Count, Certifications_Count
ORDER BY Internships_Count DESC, Certifications_Count DESC;

# Categorizing students into talent profiles
SELECT 
    Student_Id,
    Degree,
    CASE 
        WHEN Technical_Skills_Score_100 > 85 AND Communication_Skills_Score_100 > 85 THEN 'All-Rounder'
        WHEN Technical_Skills_Score_100 > 85 THEN 'Technical Specialist'
        WHEN Communication_Skills_Score_100 > 85 THEN 'Communication Specialist'
        ELSE 'Developing'
    END AS Student_Profile,
    Salary_Offered_USD
FROM student_placement_skills_2025;

# Comparing individual salary to the global average and calculating a running total
SELECT 
    Student_Id,
    Degree,
    Salary_Offered_USD,
    ROUND(AVG(Salary_Offered_USD) OVER(), 2) AS Global_Avg_Salary,
    ROUND(Salary_Offered_USD - AVG(Salary_Offered_USD) OVER(), 2) AS Difference_From_Avg,
    ROUND(SUM(Salary_Offered_USD) OVER(ORDER BY Salary_Offered_USD DESC), 2) AS Running_Total_Salary_Pool
FROM student_placement_skills_2025
WHERE Placement_Offer = 'Yes';

Tableau Link:
[Student Placement Overview](https://public.tableau.com/app/profile/chris.smith5081/viz/StudentPlacement_17715112372010/Dashboard1)

To allow for deep-dive exploration of the recruitment data, I developed an interactive Tableau dashboard centered around user-driven parameters and advanced calculated fields.

1. Dynamic User Interactivity (Parameters)
Gender Toggle: Created a Gender Parameter that allows users to filter the entire dashboard view.

Global Impact: This parameter was integrated into 3 distinct worksheets, ensuring that as a user switches views, the data remains synchronized across all KPIs.

2. Custom Calculated Fields
Developed complex Calculated Fields to bridge the gap between the parameter and the raw data.

Engineered custom calculations for Averages and Medians, providing a more robust statistical look at salary and skill scores than simple sums would allow.

3. Distribution Analysis: Circle Views
Salary vs. GPA Correlation: Leveraged a Circle View (Dot Plot) to map individual student placement success.

Why this chart? By plotting Salary against GPA levels, this visual highlights clusters of high-performers and identifies outliers—such as students with lower GPAs who secured high-tier salaries due to superior technical scores.

