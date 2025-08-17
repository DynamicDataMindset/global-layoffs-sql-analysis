# 🗂️ Global Tech Layoffs Data Cleaning (SQL)

This project focuses on **cleaning a raw dataset of global tech layoffs** using SQL.  
It was inspired by [Alex the Analyst’s SQL Data Cleaning Tutorial](https://youtu.be/4UltKCnnnTA?si=isCPIWwyxJn-pVnY).  

The goal was to take **messy real-world data**, remove inconsistencies, and prepare it for reliable **analysis and visualization**.

---

## 🚀 Skills & Tools
- **SQL (MySQL)**
- Window Functions (`ROW_NUMBER()`)
- Data Transformation & Standardization
- Handling NULLs & Blanks
- Deduplication Strategies
- Schema Management

---

## 📊 Dataset
- **Source:** `layoffs.csv` (raw dataset)  
- **Columns:**
  - `company`
  - `location`
  - `industry`
  - `total_laid_off`
  - `percentage_laid_off`
  - `date`
  - `stage`
  - `country`
  - `funds_raised_millions`

### ⚠️ Issues in Raw Data
- **Duplicates**  
- **Inconsistent formatting**  
  - `"Crypto"` vs. `"Cryptocurrency"`  
  - `"United States."` vs. `"United States"`  
- **Typos** (`compnay` instead of `company`)  
- **NULLs & blanks**  
- **Dates stored as text** instead of proper `DATE` type  

---

## 🛠️ Cleaning Process

### 🔹 Step 1: Preserve Raw Data
Always protect the original dataset by working with a **staging table**.
```sql
CREATE TABLE layoffs_staging LIKE layoffs;
INSERT INTO layoffs_staging SELECT * FROM layoffs;

🔹 Step 2: Remove Duplicates

Used a window function to detect duplicates and keep only unique rows.

ROW_NUMBER() OVER (
  PARTITION BY company, industry, total_laid_off, percentage_laid_off, date
) AS row_num


Then deleted rows where row_num > 1.

🔹 Step 3: Standardize Data

Trimmed whitespace in company names:

UPDATE layoffs_staging2 SET company = TRIM(company);


Standardized industries (Crypto% → Crypto).

Cleaned countries (United States. → United States).

Converted dates from text → proper DATE format.

🔹 Step 4: Handle NULLs

Replaced blanks with NULL.

Used self-joins to fill missing industry values where possible.

🔹 Step 5: Prune Irrelevant Data

Dropped helper column row_num.

Removed rows where both total_laid_off and percentage_laid_off were null.

🧩 Challenges & Fixes

Safe update mode error: MySQL Workbench prevented updates/deletes without a key.
✅ Fixed by temporarily disabling safe mode:

SET SQL_SAFE_UPDATES = 0;


Column typo: accidentally created compnay instead of company.
✅ Fixed using:

ALTER TABLE layoffs_staging2 CHANGE COLUMN compnay company TEXT;

✅ Results

Dataset is fully cleaned and standardized.

No duplicates, minimal nulls, consistent formats.

Ready for SQL exploration, dashboarding, or business analysis.

💡 What I Learned

Importance of staging tables to protect raw data.

Step-by-step data cleaning workflow in SQL: duplicates → standardization → null handling → pruning.

Debugging common SQL errors (safe mode, schema typos).

How clean data directly enables better analytics and visualization.

🧩 Full SQL Queries for this project with Comments 

-- ========================================================
-- 📌 DATA CLEANING PROJECT: Global Tech Layoffs (SQL)
-- Goal: Clean and standardize raw layoff dataset for analysis
-- Steps:
--   1. Remove duplicates
--   2. Standardize the data
--   3. Handle NULLs / blank values
--   4. Remove unnecessary rows & columns
-- ========================================================

-- Preview the raw data
SELECT *
FROM layoffs;

-- ========================================================
-- 1. CREATE A STAGING TABLE
--    👉 Best practice: never clean raw data directly.
--    We create a copy (layoffs_staging) and work on that.
-- ========================================================
CREATE TABLE layoffs_staging LIKE layoffs;

INSERT layoffs_staging
SELECT *
FROM layoffs;

SELECT *
FROM layoffs_staging;

-- ========================================================
-- 2. REMOVE DUPLICATES
--    👉 Using ROW_NUMBER() to detect duplicate rows based
--    on key fields (company, industry, layoffs, date, etc.)
-- ========================================================

-- Check duplicates with row numbers
SELECT *,
ROW_NUMBER() OVER(
    PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
) AS row_num
FROM layoffs_staging;

-- Create a CTE to view duplicates
WITH duplicate_cte AS (
    SELECT *,
    ROW_NUMBER() OVER(
        PARTITION BY company, industry, total_laid_off, percentage_laid_off, `date`
    ) AS row_num
    FROM layoffs_staging
) 
SELECT *
FROM duplicate_cte
WHERE row_num > 1; -- rows flagged as duplicates

-- Build a new staging table with row_num included
CREATE TABLE `layoffs_staging2` (
	`compnay` text,  -- typo included here on purpose (fixed later)
    `location` text,
    `industry` text,
    `total_laid_off` int DEFAULT NULL,
    `percentage_laid_off` text,
    `date` text,
    `stage` text,
    `country` text,
    `funds_raised_millions` int DEFAULT NULL,
    `row_num` int
);

-- Verify new table
SELECT *
FROM layoffs_staging2;

-- Insert data with row numbers for duplicates
INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER(
    PARTITION BY company, industry, total_laid_off, percentage_laid_off, `date`
) AS row_num
FROM layoffs_staging;

-- Inspect duplicates
SELECT *
FROM layoffs_staging2
WHERE row_num > 1;

-- Disable safe mode (needed to delete rows without primary key)
SET SQL_SAFE_UPDATES = 0;

-- Delete duplicates (keep only row_num = 1)
DELETE 
FROM layoffs_staging2
WHERE row_num > 1;

-- Verify cleaned table
SELECT *
FROM layoffs_staging2;

-- ========================================================
-- 3. STANDARDIZE THE DATA
--    👉 Fix typos, text inconsistencies, whitespace, etc.
-- ========================================================

-- Check columns
SHOW COLUMNS FROM layoffs_staging2;

-- Fix column name typo ("compnay" → "company")
ALTER TABLE layoffs_staging2
CHANGE COLUMN compnay company TEXT;

-- Trim extra spaces from company names
SELECT company, TRIM(company)
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET company = TRIM(company);

-- Standardize industries ("Crypto%" → "Crypto")
SELECT *
FROM layoffs_staging2
WHERE industry LIKE "Crypto%";

UPDATE layoffs_staging2
SET industry = "Crypto"
WHERE industry LIKE "Crypto%";

-- Standardize countries (remove trailing ".")
SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_staging2
ORDER BY 1;

UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';

-- Convert text dates into proper DATE format
SELECT `date`,
STR_TO_DATE(`date`, '%m/%d/%Y')
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

-- Change column type to DATE
ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;

-- ========================================================
-- 4. HANDLE NULLS / BLANKS
--    👉 Replace blanks with NULL, fill missing values where possible
-- ========================================================

-- Check rows with both layoffs fields missing
SELECT *
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

-- Replace blank industry values with NULL
UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';

-- Inspect rows with missing industries
SELECT *
FROM layoffs_staging2
WHERE industry IS NULL
OR industry = '';

-- Example: Airbnb had some NULLs → try filling from other rows
SELECT *
FROM layoffs_staging2
WHERE company = 'Airbnb';

-- Self-join to fill NULL industries using other rows of same company/location
SELECT t1.industry, t2.industry
FROM layoffs_staging2 AS t1
JOIN layoffs_staging2 AS t2
    ON t1.company = t2.company
    AND T1.location = t2.location
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;

-- Perform update to fill missing industries
UPDATE layoffs_staging2 AS t1
JOIN layoffs_staging2 AS t2
    ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL 
AND t2.industry IS NOT NULL;

-- ========================================================
-- 5. REMOVE IRRELEVANT ROWS & COLUMNS
-- ========================================================

-- Check for useless rows (both layoff fields null)
SELECT * 
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

-- Delete useless rows
DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

-- Drop helper column (row_num)
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;

-- Final cleaned dataset
SELECT * 
FROM layoffs_staging2;

-- ========================================================
-- 🎉 DATA IS NOW CLEANED & READY FOR ANALYSIS
-- ========================================================

## 📅 Next Steps  

The next step after cleaning the raw dataset is to perform **Exploratory Data Analysis (EDA)**.  
This second project is also inspired by [Alex the Analyst’s SQL EDA tutorial](https://youtu.be/QYd-RtK58VQ?si=RTRvB2Jk078SdCYg).  

Here, we use the **cleaned dataset (`layoffs_staging2`)** from the first project to explore insights and trends.  

---

# 📊 SQL Exploratory Data Analysis (EDA)  

Below is the structured SQL script with explanations for each block of analysis:  

---

### 🔹 Preview the cleaned dataset  
```sql
SELECT *
FROM layoffs_staging2;
🔹 Get maximum layoffs and highest layoff percentage
sql
Copy
Edit
SELECT MAX(total_laid_off), MAX(percentage_laid_off)
FROM layoffs_staging2;
🔹 Companies that laid off 100% of their workforce
sql
Copy
Edit
SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY total_laid_off DESC;
🔹 Same as above, ordered by funding raised
sql
Copy
Edit
SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;
🔹 Total layoffs per company
sql
Copy
Edit
SELECT company, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;
🔹 Earliest and latest layoff dates
sql
Copy
Edit
SELECT MIN(`date`), MAX(`date`)
FROM layoffs_staging2;
🔹 Total layoffs by industry
sql
Copy
Edit
SELECT industry, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;
🔹 Total layoffs by country
sql
Copy
Edit
SELECT country, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY country
ORDER BY 2 DESC;
🔹 Total layoffs per date
sql
Copy
Edit
SELECT `date`, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY `date`
ORDER BY 1 DESC;
🔹 Yearly layoffs trend
sql
Copy
Edit
SELECT YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY YEAR(`date`)
ORDER BY 1 DESC;
🔹 Layoffs grouped by funding stage
sql
Copy
Edit
SELECT stage, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY stage
ORDER BY 1 DESC;
🔹 Monthly layoffs trend (aggregated)
sql
Copy
Edit
SELECT SUBSTRING(`date`,1,7) AS `MONTH`, SUM(total_laid_off)
FROM layoffs_staging2
WHERE SUBSTRING(`date`,1,7)
GROUP BY `MONTH`
ORDER BY 1 ASC;
🔹 Rolling total of layoffs (cumulative by month)
sql
Copy
Edit
WITH Rolling_Total AS (
  SELECT SUBSTRING(`date`,1,7) AS `MONTH`, SUM(total_laid_off) AS total_off
  FROM layoffs_staging2
  WHERE SUBSTRING(`date`,1,7)
  GROUP BY `MONTH`
  ORDER BY 1 ASC
)
SELECT `MONTH`, total_off, SUM(total_off) OVER(ORDER BY `MONTH`) AS rolling_total
FROM Rolling_Total;
🔹 Company-level layoffs (totals)
sql
Copy
Edit
SELECT company, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;
🔹 Company layoffs per year
sql
Copy
Edit
SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company, YEAR(`date`)
ORDER BY 3 DESC;
🔹 Top 5 companies by layoffs each year
sql
Copy
Edit
WITH company_year (company, years, total_laid_off) AS 
(
  SELECT company, YEAR(`date`), SUM(total_laid_off)
  FROM layoffs_staging2
  GROUP BY company, YEAR(`date`)
), 
Company_Year_Rank AS 
(
  SELECT *, 
    DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
  FROM company_year
  WHERE years IS NOT NULL
) 
SELECT *
FROM Company_Year_Rank
WHERE ranking <= 5;

✅ Summary of Insights from SQL EDA

Identified companies with 100% layoffs (full shutdowns).

Observed which industries and countries were most impacted.

Tracked layoffs trends over time (monthly + cumulative).

Highlighted top companies per year with the most layoffs.

This EDA sets the stage for visualizations and storytelling dashboards in future steps.
