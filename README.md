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
