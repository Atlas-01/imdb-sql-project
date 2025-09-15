# 🎬 IMDB Top 1000 SQL Exploration

## 📌 Project Overview
This project explores the **IMDB Top 1000 Movies dataset** using **MySQL Workbench**.  
The focus is on:  
- Cleaning raw CSV data into a relational structure.  
- Running **exploratory SQL queries** to uncover insights about ratings, genres, directors, and movie trends.  
- Handling **outliers** (e.g., blockbuster movies dominating revenue/votes) to show both **industry extremes** and **typical movie patterns**.  

---

## 📂 Repository Structure
imdb-sql-project/
│
├── data/
│ └── imdb_top_1000.csv # raw dataset (or instructions to download)
│
├── sql/
│ ├── 01_create_schema.sql # schema + staging table
│ ├── 02_import_clean.sql # import + cleaning transformations
│ ├── 03_exploration_queries.sql # analysis queries
│ └── 04_outlier_handling.sql # filtering + log transform examples
│
├── visuals/ # optional graphs/exports
│
└── README.md # this file


---

## 📊 Dataset
- **Source**: [IMDB Top 1000 Movies dataset](https://www.kaggle.com/datasets)  
- **Size**: ~1000 rows × 16 columns  
- **Fields**: Title, Year, Genre, IMDB Rating, Runtime, Director, Stars, Number of Votes, Gross Revenue, etc.  

---

## 🧹 Data Cleaning Steps
1. **Staging Table:** Imported all columns as `TEXT` to avoid errors.  
2. **Typed Table:** Converted fields to appropriate data types:  
   - `Released_Year → INT`  
   - `Runtime → minutes (INT)`  
   - `IMDB_Rating → DECIMAL(3,1)`  
   - `No_of_Votes → INT`  
   - `Gross → BIGINT`  
3. **Transformations:**  
   - Removed commas from numeric fields.  
   - Extracted `Decade` from `Released_Year`.  
   - Standardized NULLs and trimmed whitespace.  

---

## 🔍 Outlier Handling
Blockbusters (e.g., *Avengers*, *Titanic*) dominate votes and gross.  
To balance analysis:  
- **Filtering:** Removed top 1% (extreme values).  
- **Log Transform:** Used `LOG10()` to compress scale and keep blockbusters visible.  

👉 This provides **two perspectives**:  
- Industry dominated by mega-hits.  
- Trends across the *majority* of movies.  

---

## 📈 Exploratory Queries (Highlights)

### 🎥 Top Movies
```sql
SELECT Series_Title, Released_Year, IMDB_Rating, No_of_Votes
FROM imdb_clean
ORDER BY IMDB_Rating DESC, No_of_Votes DESC
LIMIT 20;

📅 Ratings by Decade
SELECT Decade, ROUND(AVG(IMDB_Rating),2) AS avg_rating, COUNT(*) AS cnt
FROM imdb_clean
GROUP BY Decade
ORDER BY Decade;
📌 Shows how average ratings shift over time.

🎬 Directors with Best Average Ratings
SELECT Director, COUNT(*) AS movies, ROUND(AVG(IMDB_Rating),2) AS avg_rating
FROM imdb_clean
GROUP BY Director
HAVING movies >= 3
ORDER BY avg_rating DESC, movies DESC
LIMIT 25;

⏱ Runtime Buckets
SELECT
  CASE
    WHEN Runtime_min < 90 THEN '<90'
    WHEN Runtime_min BETWEEN 90 AND 119 THEN '90-119'
    WHEN Runtime_min BETWEEN 120 AND 149 THEN '120-149'
    ELSE '150+'
  END AS runtime_bucket,
  COUNT(*) AS cnt,
  ROUND(AVG(IMDB_Rating),2) AS avg_rating
FROM imdb_clean
GROUP BY runtime_bucket
ORDER BY cnt DESC;

-- Detect top gross and votes
SELECT Series_Title, No_of_Votes
FROM imdb_clean
ORDER BY No_of_Votes DESC
LIMIT 10;

SELECT Series_Title, Gross
FROM imdb_clean
ORDER BY Gross DESC
LIMIT 10;

-- Percentile (MySQL 8+)
SELECT
  PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY No_of_Votes) AS votes_p99,
  PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY Gross) AS gross_p99
FROM imdb_clean;

-- Filter out top 1%
SELECT Series_Title, Released_Year, IMDB_Rating, No_of_Votes, Gross
FROM imdb_clean
WHERE No_of_Votes <= 1500000
  AND Gross <= 800000000;

-- Log transform
SELECT Series_Title, IMDB_Rating,
       LOG10(No_of_Votes+1) AS log_votes,
       LOG10(Gross+1) AS log_gross
FROM imdb_clean
WHERE No_of_Votes IS NOT NULL AND Gross IS NOT NULL;





🚀 How to Reproduce

Clone this repo:

git clone https://github.com/your-username/imdb-sql-project.git
cd imdb-sql-project


Open MySQL Workbench.

Run scripts in order:

01_create_schema.sql

Import data/imdb_top_1000.csv into imdb_staging.

02_import_clean.sql

03_exploration_queries.sql

04_outlier_handling.sql
