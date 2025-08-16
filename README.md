Here’s a polished, **GitHub-ready README.md** version of your project.
I’ve styled it with markdown best practices, emojis, headers, and syntax highlighting for SQL so that it looks clean, professional, and engaging:

````markdown
# 🗂️ Global Tech Layoffs Data Cleaning (SQL)

This project focuses on **cleaning a raw dataset of global tech layoffs** using SQL.  
It was inspired by [Alex the Analyst’s SQL data cleaning tutorial](https://www.youtube.com/c/AlexTheAnalyst).

---

## 🚀 Skills & Tools
- **SQL (MySQL)**
- Window Functions
- Data Transformation
- Handling NULLs
- Standardization & Deduplication

---

## 📊 Dataset
- **Source:** `layoffs.csv` (raw)
- **Columns:**  
  `company`, `location`, `industry`, `total_laid_off`, `percentage_laid_off`,  
  `date`, `stage`, `country`, `funds_raised_millions`

### ⚠️ Issues in Raw Data
- Duplicates  
- Inconsistent formatting  
  - e.g., *"Crypto"* vs. *"Cryptocurrency"*  
  - *United States.* vs. *United States*  
- Typos  
- NULLs & blanks  
- Dates stored as text  

---

## 🛠️ Cleaning Process

### 🔹 Step 1: Preserve Raw Data
```sql
CREATE TABLE layoffs_staging LIKE layoffs;
INSERT INTO layoffs_staging SELECT * FROM layoffs;
````

👉 Work in a **staging copy** to avoid overwriting raw data.

---

### 🔹 Step 2: Remove Duplicates

Used `ROW_NUMBER()` with `PARTITION BY` across key columns.

```sql
SELECT *,
    ROW_NUMBER() OVER (
        PARTITION BY company, location, industry, total_laid_off,
                     percentage_laid_off, date, stage, country, funds_raised_millions
        ORDER BY company
    ) AS row_num
FROM layoffs_staging;
```

👉 Deleted rows where `row_num > 1` → ensures no duplicate entries.

---

### 🔹 Step 3: Standardize Data

* Trimmed whitespace:

  ```sql
  UPDATE layoffs_staging
  SET company = TRIM(company);
  ```
* Industries standardized (`Crypto% → Crypto`)
* Countries cleaned (`United States.` → `United States`)
* Dates converted to proper type:

  ```sql
  UPDATE layoffs_staging
  SET date = STR_TO_DATE(date, '%m/%d/%Y');
  ```

---

### 🔹 Step 4: Handle NULLs

* Blank industries → set to NULL
* Filled missing industries by **self-join** on same company/location

👉 Result: reduced data loss, improved completeness.

---

### 🔹 Step 5: Remove Irrelevant Data

* Dropped helper column `row_num`
* Deleted rows where both `total_laid_off` and `percentage_laid_off` were NULL

👉 Keeps only useful, analyzable data.

---

## ⚡ Challenges & Fixes

* **Safe update mode error**

  ```sql
  SET SQL_SAFE_UPDATES = 0;
  ```
* **Column typo (`compnay` instead of `company`)**

  ```sql
  ALTER TABLE layoffs_staging CHANGE COLUMN compnay company TEXT;
  ```

📌 Learned how critical schema accuracy is.

---

## ✅ Results

* Dataset **fully cleaned**
* Ready for **visualization & advanced analytics**
* **No duplicates, fewer NULLs, consistent formatting**

---

## 💡 Reflections / What I Learned

* Structured SQL workflow: **duplicates → standardization → NULLs → pruning**
* Importance of staging tables for safe experimentation
* Real-world challenges: typos, inconsistent formats, MySQL safe mode
* Strong foundation for **dashboards & business insights**

---

📌 *Next step:* Create visualizations & insights dashboards using the cleaned dataset!

```

Would you like me to also **add badges (MySQL, SQL, Data Cleaning)** and maybe a **table of contents** at the top to make it even more “GitHub-pro project” style?
```
