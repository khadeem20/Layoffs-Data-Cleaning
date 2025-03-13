# Layoffs-Data-Cleaning
A Database table containing company records of wide-wide layoffs is put through the "data cleaning process."

To follow good practise staging tables were used to adjust the data as to prevent incorrect deletion or updates to the "official" table.

--Step 1-- Remove Duplicate Records
To remove the duplicate records it was first neccessary to create some sort of identifier for them. As the table had no Id column a window function was used to add a row_num coloumn.

    SELECT *,
    ROW_NUMBER OVER(
    PARTITION BY company, industry , total_laid_off, percentage_laid_off, `date`,
    stage, country) as row_num
    from layoffs_staging;

--Step 2-- Standardize the data
The columns were reviewed and discrepancies in the company, industry, date and location columns handled.

    Update layoffs_staging2
    set company = trim(company);
    
    UPDATE layoffs_staging2
    set industry = "Crypto"
    Where industry like 'Crypto%';
    
    UPDATE layoffs_staging2
    set country = "United States"
    Where industry like 'United States%';

    UPDATE layoffs_staging2
    set `date` = STR_TO_DATE(`date`, '%m/%d/%Y') ;

    ALTER TABLE layoffs_staging
    MODIFY COLUMN `date` DATE;

--Step 3-- Remove Null or blank values

  UPDATE layoffs_staging2
  set industry = NULL
  WHERE industry = '';
  
  SELECT * FROM layoffs_staging2 t1
  JOIN layoffs_staging2 t2
    ON t1.company = t2.company
  WHERE(t1.industry is NULL)
  AND t2.industry IS NOT NULL;


  DELETE FROM layoffs_staging2
  WHERE total_laid_off is Null
  AND percentage_laid_off is Null;
  
--Step 4-- Remove any unnecessary columns

ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
