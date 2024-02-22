# Data-cleaning-using-SQL
/*
Data Cleansing Operations in SQL
*/

-- Standardizing Date Format

SELECT 
    CONVERT(DATE, Sale_Date) AS Formatted_Sale_Date
FROM 
    CleanedData.dbo.Nashville_Real_Estate;

-- Update the Sale_Date column

UPDATE 
    CleanedData.dbo.Nashville_Real_Estate
SET 
    Sale_Date = CONVERT(DATE, Sale_Date);

-- If update fails, add a new column for formatted date

ALTER TABLE 
    CleanedData.dbo.Nashville_Real_Estate
ADD 
    Sale_Date_Formatted DATE;

-- Update the newly added column with formatted dates

UPDATE 
    CleanedData.dbo.Nashville_Real_Estate
SET 
    Sale_Date_Formatted = CONVERT(DATE, Sale_Date);

-- Populating Missing Property Addresses

UPDATE 
    a
SET 
    a.Property_Address = ISNULL(a.Property_Address, b.Property_Address)
FROM 
    CleanedData.dbo.Nashville_Real_Estate a
JOIN 
    CleanedData.dbo.Nashville_Real_Estate b
ON 
    a.Parcel_ID = b.Parcel_ID
    AND a.Unique_ID <> b.Unique_ID
WHERE 
    a.Property_Address IS NULL;

-- Splitting Address into Individual Columns

ALTER TABLE 
    CleanedData.dbo.Nashville_Real_Estate
ADD 
    Address NVARCHAR(255),
    City NVARCHAR(255),
    State NVARCHAR(255);

UPDATE 
    CleanedData.dbo.Nashville_Real_Estate
SET 
    Address = SUBSTRING(Property_Address, 1, CHARINDEX(',', Property_Address) - 1),
    City = SUBSTRING(Property_Address, CHARINDEX(',', Property_Address) + 2, CHARINDEX(',', Property_Address, CHARINDEX(',', Property_Address) + 1) - CHARINDEX(',', Property_Address) - 2),
    State = SUBSTRING(Property_Address, CHARINDEX(',', Property_Address, CHARINDEX(',', Property_Address) + 1) + 2, LEN(Property_Address) - CHARINDEX(',', Property_Address, CHARINDEX(',', Property_Address) + 1) - 1);

-- Updating "Sold as Vacant" field

UPDATE 
    CleanedData.dbo.Nashville_Real_Estate
SET 
    Sold_As_Vacant = CASE 
                          WHEN Sold_As_Vacant = 'Y' THEN 'Yes'
                          WHEN Sold_As_Vacant = 'N' THEN 'No'
                          ELSE Sold_As_Vacant
                      END;

-- Removing Duplicate Records

WITH RowNumberedData AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (
            PARTITION BY Parcel_ID, Property_Address, Sale_Price, Sale_Date, Legal_Reference
            ORDER BY Unique_ID
        ) AS Row_Num
    FROM 
        CleanedData.dbo.Nashville_Real_Estate
)
DELETE FROM 
    RowNumberedData
WHERE 
    Row_Num > 1;

-- Removing Unused Columns

ALTER TABLE 
    CleanedData.dbo.Nashville_Real_Estate
DROP COLUMN 
    Owner_Address, Tax_District, Property_Address, Sale_Date;
