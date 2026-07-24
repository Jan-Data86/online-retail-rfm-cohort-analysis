-- =============================================
-- 1. Create table for cleaned data (imported from Python cleaning pipeline)
-- =============================================

CREATE TABLE online_retail_clean_py (
    Invoice VARCHAR(20),
    StockCode VARCHAR(20),
    Description VARCHAR(255),
    Quantity INT,
    InvoiceDate DATETIME,
    Price DECIMAL(10,2),
    CustomerID INT,
    Country VARCHAR(100)
);

-- =============================================
-- 2. Load cleaned CSV (df_clean, exported from pandas) into the table.
-- CustomerID empty strings are converted to real NULL via NULLIF,
-- since pandas exports missing values as empty strings, not SQL NULL.
-- =============================================

LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 9.7/Uploads/df_clean_export.csv'
INTO TABLE online_retail_clean_py
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(Invoice, StockCode, Description, Quantity, @InvoiceDate, Price, @CustomerID, Country)
SET
    InvoiceDate = STR_TO_DATE(@InvoiceDate, '%Y-%m-%d %H:%i:%s'),
    CustomerID = NULLIF(@CustomerID, '');

-- =============================================
-- 3. Top 10 products by total revenue (Quantity * Price, summed).
-- Cancelled orders (Invoice starting with 'C') are intentionally not filtered out:
-- an order and its cancellation net out to zero on summation, so the result
-- already reflects net revenue per product.
-- =============================================

SELECT 
    StockCode, 
    Description,
    SUM(Quantity * Price) AS Total_Revenue
FROM online_retail_clean_py
GROUP BY StockCode, Description
ORDER BY Total_Revenue DESC
LIMIT 10;

-- =============================================
-- 4. Market basket analysis: top 10 product pairs most frequently bought together.
-- Self-join on Invoice; a.StockCode < b.StockCode avoids both self-pairs (A-A)
-- and mirrored duplicates (A-B / B-A counted twice).
-- =============================================

SELECT
    a.StockCode, 
    b.StockCode,
    COUNT(*) AS count
FROM online_retail_clean_py AS a
JOIN online_retail_clean_py AS b 
    ON a.Invoice = b.Invoice
WHERE a.StockCode < b.StockCode
GROUP BY a.StockCode, b.StockCode
ORDER BY count DESC
LIMIT 10;

-- =============================================
-- 5. Top 10 products by cancellation rate (share of transactions with
-- Invoice starting with 'C'). HAVING COUNT(*) >= 20 filters out products
-- with too few orders to be statistically meaningful.
-- =============================================

SELECT
	StockCode,
    COUNT(*),
    SUM(CASE 
			WHEN Invoice LIKE 'C%' THEN 1 
			ELSE 0
		END) AS Cancels,
	SUM(CASE 
			WHEN Invoice LIKE 'C%' THEN 1 
			ELSE 0
		END) / COUNT(*) AS Cancel_Rate
FROM online_retail_clean_py
GROUP BY StockCode
HAVING COUNT(*) >= 20	
ORDER BY Cancel_Rate DESC
LIMIT 10

-- =============================================
-- 6. Investigating the CHERRY LIGHTS product line (StockCode prefix 79323),
-- which dominates the cancellation-rate results above. Checking for
-- inconsistent/duplicate Description values across StockCodes.
-- =============================================

SELECT DISTINCT StockCode, Description
FROM online_retail_clean_py
WHERE StockCode LIKE '79323%'
