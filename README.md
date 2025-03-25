# Data-Cleaning-with-SQL

### Project overview

This data analysis project aims to provude insights into the sales performance of an e-commerce company over the past year. By analyzing various aspects of the sales data, we seek to identify trends, make data-driven recommendation, and gain a deeper understanding of the company's performance.

### Data Sources

Sales Data: The primary dataset used for this analysis is the "layoffs.csv" file, containing detailed information about each sale made by the company.

### Tools

- SQL Server

 ### Data Cleaning/Preparation

 1. Data Loading and Inspection
 2. Creation of a duplicate table
 3. Removal of duplicates
 4. Standardization of data
 5. POpulation of Null/Blank values
 6. Removal of unnecessary columns/ rows

#### 1. Create a duplicate table

```sql
SELECT*
FROM layoffs;

CREATE TABLE layoffs_staging
LIKE layoffs;

SELECT*
FROM layoffs_staging;

INSERT layoffs_staging
SELECT *
FROM layoffs;
```

#### 2. Removing Duplicated Values

```sql
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, industry, total_laid_off, percentage_laid_off, 'date') AS row_num
FROM layoffs_staging
;

WITH duplicate_cte AS
(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
)
SELECT *
FROM duplicate_cte
WHERE row_num >1;

SELECT*
FROM layoffs_staging
WHERE company = 'casper';


CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

SELECT*
FROM layoffs_staging2
WHERE row_num > 1;

INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;

DELETE
FROM layoffs_staging2
WHERE row_num > 1;

SELECT *
FROM layoffs_staging2;
```
#### 3. Standardize data

Taking off white spaces at the beginning and end of a word using TRIM

```sql
SELECT company, TRIM(company)
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET company = TRIM(company);
```

Order data in the 'Industry' column alphabetically to check the data

```sql
SELECT DISTINCT industry
FROM layoffs_staging2
ORDER BY 1;

SELECT*
FROM layoffs_staging2
WHERE industry LIKE 'Crypto%';

-- crypto is appearing in diffrent manner hence we should make it all to be called crypto.
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```
Checking data in the 'location' column.

```sql
SELECT DISTINCT country
FROM layoffs_staging2
ORDER BY 1;

-- Country Column
SELECT*
FROM layoffs_staging2
WHERE country LIKE 'United States'
ORDER BY 1;
-- United states has a dot at the end and should be removed
SELECT DISTINCT country, TRIM( TRAILING '.' FROM country)
FROM layoffs_staging2
ORDER BY 1;

UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';
```

Formatting Date Column to the standard formart in mySQL

```sql
-- Date Column
SELECT `date`,
STR_TO_DATE(`date`, '%m/%d/%Y')
FROM layoffs_staging;

UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');
```

Change the data type of 'Date' colunm from Text to Date.

```sql
SELECT `date`
FROM layoffs_staging2;

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```

#### 4. Populate Null/ Blank values

```sql
-- Null values

SELECT*
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';

SELECT*
FROM layoffs_staging2
WHERE industry IS NULL
OR industry = '';

SELECT*
FROM layoffs_staging2
WHERE company = 'Airbnb';

SELECT*
FROM layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
    AND t1.location = t2.location
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;
```
