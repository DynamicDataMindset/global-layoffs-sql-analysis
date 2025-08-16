1. Project Overview

This project cleans a raw dataset of global tech layoffs.

Inspired by Alex the Analyst’s SQL data cleaning tutorial.

Skills: MySQL, window functions, data transformation, handling nulls, standardization.

2. Dataset

Source: layoffs.csv (raw).

Columns include: company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions.

Issues in raw data:

Duplicates

Inconsistent formatting (e.g., "Crypto" vs. "Cryptocurrency", United States. vs. United States)

Typos

Nulls & blanks

Date stored as text instead of proper date type

3. Cleaning Process

Here we explain your SQL step-by-step (with reasoning).

🔹 Step 1: Preserve Raw Data
CREATE TABLE layoffs_staging LIKE layoffs;
INSERT layoffs_staging SELECT * FROM layoffs;


👉 Keep original table intact to avoid overwriting raw data. Work in a staging copy.

🔹 Step 2: Remove Duplicates

Used ROW_NUMBER() with PARTITION BY across key columns.

Duplicates = any row where all attributes are the same.

Insert into layoffs_staging2 with row_num.

Delete rows where row_num > 1.
👉 Ensures no duplicate entries remain.

🔹 Step 3: Standardize Data

Company names → trimmed whitespace (TRIM(company)).

Industries → standardized categories (Crypto% → Crypto).

Countries → removed trailing periods (United States. → United States).

Dates → converted string → DATE type with STR_TO_DATE.
👉 Improves consistency for querying & analysis.

🔹 Step 4: Handle Nulls

Blank industries → set to NULL.

Filled missing industry values by self-joining rows from same company/location that had industry populated.
👉 Reduces data loss, improves completeness.

🔹 Step 5: Remove Irrelevant Columns/Rows

Removed helper column row_num.

Deleted rows where both total_laid_off and percentage_laid_off were null.
👉 Keeps only useful, analyzable data.

4. Challenges & Fixes

Safe update mode error → MySQL Workbench protection against accidental mass deletes. Fixed by disabling safe mode with:

SET SQL_SAFE_UPDATES = 0;


Column typo (compnay instead of company) → fixed with:

ALTER TABLE layoffs_staging2 CHANGE COLUMN compnay company TEXT;


Learned how important schema accuracy is.

5. Results

✅ Dataset fully cleaned.

✅ Ready for visualization, aggregation, or advanced analytics.

✅ Better data integrity, fewer nulls, no duplicates.

6. Reflections / What I Learned

Practical SQL cleaning workflow: duplicates → standardization → nulls → pruning.

Importance of staging tables for safety.

Real-world challenge: safe mode errors, typos, inconsistent text formats.

Foundation for data analytics dashboards and business insights.
