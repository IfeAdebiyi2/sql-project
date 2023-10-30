What issues will you address by cleaning the data?
  The following are the issues I addressed by cleaning the data:
1. Appropriate data types for all columns to ensure the data consistency in format and structure of the tables.

2. Reduction of data redundancy by removing duplicate data, this helps to make sure that each piece of the dataset represents a distinct or unique entity.

3. Eliminated nulls where applicable by filling in values.




Queries:
Below, provide the SQL queries you used to clean your data.

1. Change of Data Types
- All table columns were created in the 'varchar' datatype when importing the csv. files into the database
- Then, changed data type of each column to match the format of values contained through rows using the following syntax:
			
			ALTER TABLE table_name
			ALTER COLUMN column_name SET DATA TYPE data_type
				USING column_name::data_type

*To note: Multiple columns in the same table were altered in one single query by entering each one after the other* 

-The following were the specific queries used in each table:

a)TABLE: all_sessions

ALTER TABLE all_sessions
ALTER COLUMN fullvisitorid TYPE numeric
	USING fullvisitorid::numeric;
ALTER COLUMN totaltransactionrevenue SET DATA TYPE int
	USING totaltransactionrevenue::integer;
ALTER COLUMN transactions SET DATA TYPE int
	USING transactions::integer;
ALTER COLUMN pageviews SET DATA TYPE int
	USING pageviews::integer,
ALTER COLUMN sessionqualitydim SET DATA TYPE int
	USING sessionqualitydim::integer;
ALTER COLUMN visitid SET DATA TYPE int
	USING visitid::integer,
ALTER COLUMN productrefundamount SET DATA TYPE int
	USING productrefundamount::integer,
ALTER COLUMN productquantity SET DATA TYPE int
	USING productquantity::integer,
ALTER COLUMN productprice SET DATA TYPE int
	USING productprice::integer,
ALTER COLUMN productrevenue SET DATA TYPE int
	USING productrevenue::integer,
ALTER COLUMN itemquantity SET DATA TYPE int
	USING itemquantity::integer,
ALTER COLUMN itemrevenue SET DATA TYPE int
	USING itemrevenue::integer,
ALTER COLUMN transactionrevenue SET DATA TYPE int
	USING transactionrevenue::integer,
ALTER COLUMN ecommerceaction_type SET DATA TYPE int
	USING ecommerceaction_type::integer,
ALTER COLUMN ecommerceaction_step SET DATA TYPE int
	USING ecommerceaction_step::integer,
ALTER COLUMN date SET DATA TYPE date
	USING date::date

- To convert the "time" and "timeonsite" columns, I converted to numeric first, and then to the time datatype
ALTER TABLE all_sessions
ALTER COLUMN "time" SET DATA TYPE numeric
	USING "time"::numeric;
ALTER COLUMN timeonsite SET DATA TYPE numeric
	USING timeonsite::numeric;

ALTER TABLE all_sessions
ALTER COLUMN "time" SET DATA TYPE time
	USING '00:00'::time + (time || 'seconds')::interval;
ALTER COLUMN timeonsite SET DATA TYPE time
	USING '00:00'::time + (timeonsite || 'seconds')::interval;


b) TABLE: analytics

ALTER TABLE analytics
ALTER COLUMN visitnumber TYPE int
	USING visitnumber::integer,
ALTER COLUMN visitid TYPE int
	USING visitid::integer,
ALTER COLUMN visitstarttime TYPE int
	USING visitstarttime::integer,
ALTER COLUMN fullvisitorid TYPE numeric
	USING fullvisitorid::numeric,
ALTER COLUMN userid TYPE int
	USING userid::integer,
ALTER COLUMN date TYPE date
	USING date::date,
ALTER COLUMN pageviews TYPE int
	USING pageviews::integer,
ALTER COLUMN timeonsite TYPE numeric
	USING timeonsite::numeric,
ALTER COLUMN units_sold TYPE int
	USING units_sold::integer,
ALTER COLUMN bounces TYPE int
	USING bounces::integer,
ALTER COLUMN revenue TYPE numeric
	USING revenue::numeric,
ALTER COLUMN unit_price TYPE numeric
	USING unit_price::numeric;

c) TABLE: products

ALTER TABLE products
ALTER COLUMN orderedquantity TYPE int
	USING orderedquantity::integer,
ALTER COLUMN stocklevel TYPE int
	USING stocklevel::integer,
ALTER COLUMN restockingleadtime TYPE int
	USING restockingleadtime::integer,
ALTER COLUMN sentimentscore TYPE float
	USING sentimentscore::float,
ALTER COLUMN sentimentmagnitude TYPE float
	USING sentimentmagnitude::float;

d) TABLE: sales_report

ALTER TABLE sales_report
ALTER COLUMN total_ordered TYPE int
	USING total_ordered::integer,
ALTER COLUMN stocklevel TYPE int
	USING stocklevel::integer,
ALTER COLUMN restockingleadtime TYPE int
	USING restockingleadtime::integer,
ALTER COLUMN sentimentscore TYPE float
	USING sentimentscore::float,
ALTER COLUMN sentimentmagnitude TYPE float
	USING sentimentmagnitude::float,
ALTER COLUMN ratio TYPE float
	USING ratio::float;

e) TABLE: sales_by_sku

ALTER TABLE sales_by_sku
ALTER COLUMN total_ordered TYPE int
	USING total_ordered::integer;


2. Removing duplicate and redundant data and creating temporary tables for all tables

SELECT fullvisitorid, COUNT(*) AS count
FROM all_sessions
GROUP BY fullvisitorid
HAVING COUNT(*) > 1;

- Visitor Id should be unique, so there should not be duplicates

a) TABLE: all_sessions

CREATE TABLE temp_all_sessions AS
(SELECT DISTINCT fullvisitorid, channelgrouping, time, pageviews, date, visitid, type, productprice, productsku, v2productname, 
	(CASE
		WHEN country = '(not set)' THEN 'N/A'
		ELSE country
		END) AS country,
	(CASE
		WHEN city = '(not set)' THEN 'N/A'
		WHEN city = 'not available in demoset' THEN 'N/A'
		ELSE city
		END) AS city,
	(CASE
		WHEN v2productcategory = '(not set)' THEN 'N/A'
		ELSE v2productcategory
		END) AS v2productcategory,
	(CASE
		WHEN timeonsite IS NULL THEN '00:00:00'::TIME
		ELSE timeonsite
		END) AS timeonsite,
	(CASE
		WHEN productvariant = '(not set)' THEN 'N/A'
		ELSE productvariant
		END) AS productvariant,
	(CASE
		WHEN currencycode IS NULL THEN 'USD'
		ELSE currencycode
		END) AS currencycode,
	(CASE
		WHEN pagetitle IS NULL THEN 'N/A'
		ELSE pagetitle
		END) AS pagetitle
FROM all_sessions
);

b) TABLE: analytics

CREATE TABLE temp_analytics AS
(SELECT row_number() over() as distinct_id, visitid, date, fullvisitorid, channelgrouping, socialengagementtype,
	(CASE
		WHEN units_sold IS NULL THEN '0'
		ELSE units_sold
		END) AS units_sold,
	(CASE
		WHEN pageviews IS NULL THEN '0'
		ELSE pageviews
		END) AS pageviews,
	(CASE
		WHEN timeonsite IS NULL THEN '00:00:00'::TIME
		ELSE timeonsite
		END) AS timeonsite,
	(CASE
		WHEN bounces IS NULL THEN '0'
		ELSE bounces
		END) AS bounces,
	(CASE
		WHEN revenue IS NULL THEN '0'
		ELSE revenue
		END) AS revenue,
(unit_price/1000000) AS unit_price
FROM analytics
);

c) TABLE: products

CREATE TABLE temp_products AS
(SELECT row_number() over() as distinct_id, sku AS productsku, name, orderedquantity, stocklevel,restockingleadtime, 
 	(CASE 
	 	WHEN sentimentscore IS NULL THEN '0'
	 	ELSE sentimentscore
	 	END) AS sentimentscore,
 	(CASE
		WHEN sentimentmagnitude IS NULL THEN '0'
		ELSE sentimentmagnitude
		END) AS sentimentmagnitude
FROM products
);

d) TABLE: sales_reports

CREATE TABLE temp_sales_report AS
(SELECT row_number() over() as distinct_id, productsku, total_ordered, name, stocklevel,restockingleadtime, sentimentscore, sentimentmagnitude,
 	(CASE
		WHEN ratio IS NULL THEN '0'
		ELSE ratio
		END) AS ratio
FROM sales_report
);

e) TABLE: sales_by_sku

CREATE TABLE temp_sales_by_sku AS
(SELECT row_number() over() as distinct_id, productsku, total_ordered
FROM sales_by_sku
);