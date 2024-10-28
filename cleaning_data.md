What issues will you address by cleaning the data?
1. unit_price column of the analytics table 
2. duplicate rows in all_sessions
3. replacing values with average in analytics
4. missing values in the product table. Adding missing sku's that are not present in the product table but are present in sales_by_sku. This allows sku to be the foreign key in the sales_by_sku table.


Queries:
Below, provide the SQL queries you used to clean your data.
1. UPDATE analytics
SET unit_price= unit_price/1000000
2. all_session-> as my primary keys in this table are fullvisitorid and visitid, I am going to check for duplicate rows and remove them. I first find all the duplicate rows using: SELECT a.*
FROM all_sessions a
JOIN (
SELECT fullvisitorid, visitid
FROM all_sessions
GROUP BY fullvisitorid, visitid
HAVING count(*) > 1
) as duplicates
ON a.fullvisitorid = duplicates.fullvisitorid
AND a.visitid = duplicates.visitid;
Then remove them using: DELETE FROM all_sessions
WHERE (fullvisitorid, visitid) IN (
SELECT fullvisitorid, visitid
FROM all_sessions
GROUP BY fullvisitorid, visitid
HAVING count(*) > 1
); This allows me to make fullvisitorid and visitid as my primary keys. 


3. The rows with the same visitid and date have different unitprice so i will take the average of all the unitprice with the same visitid and date so that no two rows have the same visitid and date. 
SELECT a.*
FROM analytics a
JOIN (
SELECT visitid, date
FROM analytics
GROUP BY visitid, date
HAVING count(*) > 1
) as duplicates
ON a.visitid = duplicates.visitid
AND a.date = duplicates.date;

CREATE TEMP TABLE average_unit_price as
WITH avgunitprice as (
SELECT visitid, date, avg(unit_price) as avg_unit_price 
FROM analytics
GROUP BY visitid, date
HAVING count(*) > 1
)
SELECT visitid, date, avg_unit_price
FROM avgunitprice;
DELETE FROM analytics
WHERE (visitid, date) IN (
SELECT visitid, date
FROM analytics
GROUP BY visitid, date
HAVING count(*) > 1
);
INSERT INTO analytics (visitid, date, unit_price)
SELECT visitid, date, avg_unit_price
FROM average_unit_price;

4. INSERT INTO products (sku, name, orderedquantity, stocklevel, restockingleadtime, sentimentscore, sentimentmagnitude)
SELECT DISTINCT productsku, NULL as name, NULL::integer as orderedquantity, NULL::integer as stocklevel, NULL::integer as restockingleadtime, NULL::numeric as sentimentscore, NULL::numeric as sentimentmagnitude
FROM sales_by_sku
WHERE productsku NOT IN (SELECT sku FROM products);
