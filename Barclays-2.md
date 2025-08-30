# Barclays Customer Churn Analysis: Retention Gap Insights

## Introduction
This project focuses on analyzing customer churn at Barclays, a leading global financial institution. Customer churn, or the rate at which customers discontinue their relationship with the bank, is a critical metric for financial service providers. High churn rates can lead to significant revenue loss, increased acquisition costs, and diminished market share. In this analysis, I leveraged a dataset representing Barclays' customer base to uncover patterns, identify risk factors, and derive actionable insights for improving customer retention strategies.

The project demonstrates my skills in data analysis, SQL querying, and visualization using open-source tools. By examining demographic, behavioral, and financial attributes, I aimed to highlight "retention gaps"—areas where the bank could intervene to reduce churn and enhance customer loyalty.

## Objective
The primary objective of this analysis was to:

- Identify key factors contributing to customer churn at Barclays.
- Quantify the impact of variables such as age, tenure, product usage, geography, and activity status on churn rates.
- Provide data-driven recommendations to bridge retention gaps, such as targeted marketing campaigns, personalized retention programs, or engagement initiatives.
- Demonstrate proficiency in handling relational databases, writing efficient SQL queries, and creating insightful dashboards for stakeholder communication.

Ultimately, the insights from this project can help Barclays (or similar institutions) reduce churn by 10-20% through proactive measures, based on industry benchmarks.

## Technologies and Tools Used
- **Database Management:** MySQL (for storing, querying, and manipulating the dataset).  
- **Data Visualization and Dashboarding:** Metabase (for creating interactive dashboards, charts, and reports to visualize churn patterns).  
- **Programming Languages:** SQL (for data extraction and analysis).  
- **Other Tools:** Git for version control; Markdown for documentation; and basic ETL processes handled via MySQL Workbench for data import and cleaning.

No advanced machine learning libraries were used, as the focus was on exploratory data analysis (EDA) and SQL-based insights. The project is fully reproducible with the provided SQL scripts and Metabase configuration files (available in the repository).

## About the Database
The dataset used in this project is a simulated Barclays customer churn dataset, inspired by publicly available banking churn datasets (e.g., from Kaggle). It consists of approximately 10,000 customer records stored in a MySQL database. The schema includes a single table named **customer** with the following key columns:

- `customer_id`: Unique identifier for each customer (INT, PRIMARY KEY).  
- `credit_score`: Customer's credit score (INT).  
- `country`: Customer's location (e.g., France, Spain, Germany) (VARCHAR).  
- `gender`: Customer's gender (VARCHAR).  
- `age`: Customer's age (INT).  
- `tenure`: Number of years as a customer (INT).  
- `balance`: Account balance (DECIMAL).  
- `products_number`: Number of products held (e.g., loans, credit cards) (INT).  
- `credit_card`: Whether the customer has a credit card (BOOLEAN).  
- `active_member`: Whether the customer is active (BOOLEAN).  
- `estimated_salary`: Estimated annual salary (DECIMAL).  
- `churn`: Churn indicator (1 if churned, 0 otherwise) (BOOLEAN).  

The database was hosted locally on MySQL Server 8.0. Data was imported via CSV using MySQL Workbench, with basic cleaning steps to handle null values and ensure data types consistency. The total database size is under 1MB, making it efficient for analysis.  

---

## Process (CRISP-DM)
1. **Business Understanding**: Reviewed banking industry churn challenges and defined objectives.  
2. **Data Understanding**: Explored the dataset schema, checked for anomalies (`DESCRIBE customer;`, `SELECT COUNT(*) FROM customer;`) and performed initial EDA.  
3. **Data Preparation**: Cleaned data in MySQL (e.g., removing duplicates).  
4. **Modeling/Analysis**: Wrote SQL queries to answer specific questions, aggregated data, and calculated metrics like churn rates, revenue loss, and segments.  
5. **Evaluation**: Visualized results in Metabase (bar charts, heatmaps, dashboards).  
6. **Deployment**: Created dashboards in Metabase and exported insights; documented for GitHub.  

Indexes were applied on frequently queried columns like `age` and `country`.

---

## Questions Analyzed with Queries

### Question 1: Overall Churn Rate
```sql
SELECT ROUND(100.0 * SUM(churn) / COUNT(*), 2) AS churn_rate FROM customer;
```
- **Result:** 20.37% churn rate.  
- **Visualization:** Single-value card.  

### Question 2: Estimated Revenue Loss from Churned Customers
```sql
SELECT CONCAT('USD ', ROUND(SUM(balance) / 1000000, 2), ' million') AS revenue_loss FROM customer WHERE churn = 1;
```
- **Result:** USD 185.59 million.  
- **Visualization:** Gauge chart.  

### Question 3: Avg. Revenue Loss Per Churned Customer
```sql
SELECT CONCAT('USD ', ROUND(AVG(balance), 2), '/Customer') AS avg_revenue_loss FROM customer WHERE churn = 1;
```
- **Result:** USD 91,108.54.  
- **Visualization:** Single-value card.  

### Question 4: Highest Churn Risk Region
```sql
WITH country_churn AS (
    SELECT country, ROUND(100.0 * SUM(churn) / COUNT(*), 2) AS churn_rate
    FROM customer GROUP BY country
)
SELECT country, churn_rate FROM country_churn ORDER BY churn_rate DESC;
```
- **Result:** Germany (32.44%), Spain (16.67%), France (16.15%).  
- **Visualization:** Bar chart.  

### Question 5: Most Vulnerable Segment
*(Age × Activity Status)*  
```sql
SELECT CASE WHEN age < 30 THEN 'Young (<30)'
            WHEN age BETWEEN 30 AND 50 THEN 'Middle-Aged (30-50)'
            ELSE 'Senior (>50)' END AS age_group,
       CASE WHEN active_member = 0 THEN 'Inactive' ELSE 'Active' END AS activity_status,
       COUNT(*) AS total_customers, SUM(churn) AS churned_customers,
       ROUND(100.0 * SUM(churn) / COUNT(*), 2) AS churn_rate
FROM customer
GROUP BY age_group, activity_status
ORDER BY age_group, activity_status;
```
- **Result:** Inactive seniors churn at 84.94%.  
- **Visualization:** Heatmap.  

### Question 6: Highest Product Churn Rate
```sql
SELECT products_number, COUNT(*) AS total_customers, SUM(churn) AS churned_customers,
       ROUND(SUM(churn) * 100.0 / COUNT(*), 2) AS churn_rate,
       AVG(balance) AS avg_balance, AVG(estimated_salary) AS avg_salary
FROM customer
GROUP BY products_number
HAVING COUNT(*) > 30
ORDER BY products_number;
```
- **Result:** 4 products → 100% churn, 3 products → 82.71%.  
- **Visualization:** Line chart.  

### Question 7: Churn Rate by Key Demographics
```sql
SELECT COALESCE(country, 'All Countries') AS Country,
       COALESCE(gender, 'All Genders') AS Gender,
       COUNT(*) AS Total_Customers,
       SUM(churn) AS Churned_Customers,
       ROUND(SUM(churn) * 100.0 / COUNT(*), 2) AS Churn_Rate_Percent
FROM customer
GROUP BY country, gender WITH ROLLUP
HAVING (country IS NOT NULL AND gender IS NOT NULL)
    OR (country IS NOT NULL AND gender IS NULL)
    OR (country IS NULL AND gender IS NOT NULL)
    OR (country IS NULL AND gender IS NULL)
ORDER BY country, gender;
```
- **Result:** Germany female churn rate = 37.55%, males 27.81%.  
- **Visualization:** Pivot table.  

### Question 8: Tenure Analysis: Churn Risk by Tenure Band
```sql
WITH tenure_bands AS (
  SELECT customer_id, churn, tenure,
         CASE WHEN tenure <= 2 THEN '0-2 Years'
              WHEN tenure <= 4 THEN '3-4 Years'
              WHEN tenure <= 6 THEN '5-6 Years'
              WHEN tenure <= 8 THEN '7-8 Years'
              ELSE '9+ Years' END AS tenure_band
  FROM customer
)
SELECT tenure_band AS Tenure_Range,
       COUNT(*) AS Total_in_Band,
       SUM(churn) AS Churned_in_Band,
       ROUND(AVG(churn) * 100, 2) AS Churn_Rate_Percent
FROM tenure_bands
GROUP BY tenure_band
ORDER BY MIN(tenure);
```
- **Result:** Churn consistent ~18–21%.  
- **Visualization:** Bar chart.  

### Question 9: Low-Engagement Segment
```sql
SELECT country, gender, AVG(age) AS avg_age, AVG(tenure) AS avg_tenure,
       COUNT(*) AS total_customers, SUM(churn) AS churned_customers,
       ROUND(SUM(churn) * 100.0 / COUNT(*), 2) AS churn_rate
FROM customer
WHERE balance = 0 AND credit_card = 0 AND products_number = 1
GROUP BY country, gender
ORDER BY churn_rate DESC;
```
- **Result:** Spanish females → 48.65% churn.  
- **Visualization:** Table.  

### Question 10: Customer Lifetime Value (CLV) Proxy
```sql
WITH clv_data AS (
    SELECT customer_id, churn,
           (tenure * balance * products_number) AS clv_proxy,
           NTILE(4) OVER (ORDER BY (tenure * balance * products_number)) AS clv_quartile
    FROM customer
    WHERE balance > 0
)
SELECT clv_quartile, COUNT(*) AS total_customers, SUM(churn) AS churned_customers,
       ROUND(SUM(churn) * 100.0 / COUNT(*), 2) AS churn_rate,
       AVG(clv_proxy) AS avg_clv, MIN(clv_proxy) AS min_clv, MAX(clv_proxy) AS max_clv
FROM clv_data
GROUP BY clv_quartile
ORDER BY clv_quartile;
```
- **Result:** Churn rate ~22–25% across quartiles.  
- **Visualization:** Scatter plot.  

### Question 11: Country-Specific Active Member Paradox
```sql
SELECT country, active_member, COUNT(*) AS total_customers, SUM(churn) AS churned_customers,
       ROUND(SUM(churn) * 100.0 / COUNT(*), 2) AS churn_rate,
       ROUND(SUM(churn) * 100.0 / COUNT(*), 2) - 
        (SELECT ROUND(AVG(churn) * 100, 2) FROM customer c2 WHERE c2.country = customer.country) AS deviation_from_country_avg
FROM customer
GROUP BY country, active_member
ORDER BY country, active_member;
```
- **Result:** In Germany, inactive = 41.08%, active = 23.72%.  
- **Visualization:** Grouped bar chart.  

### Question 12: Balance Quartile Analysis
```sql
WITH balance_quartiles AS (
    SELECT customer_id, churn, balance,
           NTILE(4) OVER (ORDER BY balance) AS balance_quartile
    FROM customer
    WHERE balance > 0
)
SELECT balance_quartile, COUNT(*) AS total_customers, SUM(churn) AS churned_customers,
       ROUND(SUM(churn) * 100.0 / COUNT(*), 2) AS churn_rate,
       MIN(balance) AS min_balance, MAX(balance) AS max_balance, AVG(balance) AS avg_balance
FROM balance_quartiles
GROUP BY balance_quartile
ORDER BY balance_quartile;
```
- **Result:** Quartile 2 highest churn at 27.38%.  
- **Visualization:** Box plot.  

---

## Results and Outcomes
- **Key Insights:**  
  - Germany has the highest churn rate (32.44%).  
  - Inactive seniors are the most vulnerable group (84.94%).  
  - Customers with 4 products have **100% churn**.  
  - Low-engagement females churn up to **48.65%**.  
  - Churn remains fairly constant across tenure bands (~18–21%).  
  - Total revenue loss: **USD 185.59M**, avg. loss per churned customer: **USD 91K**.  

- **Recommendations:**  
  - Targeted campaigns in Germany.  
  - Re-engagement for inactive customers.  
  - Simplify product offerings.  
  - Focus retention on high-CLV segments.  

- **Expected Impact:** 15–20% churn reduction possible.  

---

## Learnings and Challenges
- **Learnings:** Advanced SQL (CTEs, NTILE, ROLLUP), MySQL–Metabase integration, multi-dimensional churn analysis.  
- **Challenges:** SQL query optimization, handling Metabase limitations, filtering zero-balance customers.  
- **Future Work:** Add predictive ML models for churn forecasting.  
