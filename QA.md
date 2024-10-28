What are your risk areas? Identify and describe them.
1. Duplicates: Removing duplicates without checking that all columns are the same value can lead to loss of data if some duplicates contain different information.
2. Replacing values with average values for the unit_price as all the columns were the same except the unit_price. This may result in misleading trends or patterns. Also, the outliers may heavily affect the value. 
3. adding rows in the products table to make sure it contained all the sku numbers. This may lead to unexpected data types.

QA Process:
Describe your QA process and include the SQL queries used to execute it.
1. Find duplicate rows: SELECT a.*
FROM all_sessions a
JOIN (
SELECT fullvisitorid, visitid
FROM all_sessions
GROUP BY fullvisitorid, visitid
HAVING COUNT(*) > 1
) as duplicates
ON a.fullvisitorid = duplicates.fullvisitorid
AND a.visitid = duplicates.visitid

then i confirmed the duplicate rows had the same data and then removed duplicate rows: DELETE FROM all_sessions
WHERE (fullvisitorid, visitid) IN (
SELECT fullvisitorid, visitid
FROM all_sessions
GROUP BY fullvisitorid, visitid
HAVING COUNT(*) > 1
)

2. First i found the duplicate rows: 
SELECT a.*
FROM analytics a
JOIN (
SELECT visitid, date
FROM analytics
GROUP BY visitid, date
HAVING COUNT(*) > 1
) as duplicates
ON a.visitid = duplicates.visitid
AND a.date = duplicates.date

then i created a temp table for merging the rows by taking the unit_price average 
CREATE TEMP TABLE average_unit_price as
WITH avgunitprice as (
SELECT visitid, date, avg(unit_price) as avg_unit_price 
FROM analytics
GROUP BY visitid, date
HAVING COUNT(*) > 1
)
SELECT visitid, date, avg_unit_price
FROM avgunitprice

then i deleted the duplicate rows: DELETE FROM analytics
WHERE (visitid, date) IN (
SELECT visitid, date
FROM analytics
GROUP BY visitid, date
HAVING COUNT(*) > 1
)
and lastly, inserted the merge results back into the table
INSERT INTO analytics (visitid, date, unit_price)
SELECT visitid, date, avg_unit_price
FROM average_unit_price

3. Adding sku numbers into the products table, i made sure the added values matched the data type so i used the cast 
INSERT INTO products (sku, name, orderedquantity, stocklevel, restockingleadtime, sentimentscore, sentimentmagnitude)
SELECT DISTINCT productsku, NULL as name, NULL::integer as orderedquantity, NULL::integer as stocklevel, NULL::integer as restockingleadtime, NULL::numeric as sentimentscore, NULL::numeric as sentimentmagnitude
FROM sales_by_sku
WHERE productsku NOT IN (SELECT sku FROM products)



 
