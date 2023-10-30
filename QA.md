What are your risk areas? Identify and describe them.

Some of my risk areas were the following:
1. Data Inconsstency - Incomplete/Missing data in some columns leading to null values
	Null values skew the result dataset and may lead to inaccurate deductions.

2. Quality of data and data cleaning process.
	Poor quality of data affects the data cleaning process ,and even with an efficient database system may still result in poor results.

3. Potential loss of data when importing dataset files and uploading tables in DB

4. Not writing clean and efficient SQL queries.
	Being completely new to the world of coding and just learning to write clean queries greatly affected how effectively I was able to clean and extrapolate results and conclusions from the dataset

QA Process:
Describe your QA process and include the SQL queries used to execute it.

1. I checked for missing values and identifies columns with NULL or empty values.

SELECT fullvisitorid
FROM all_sessions
WHERE fullvisitorid IS NOT null;


2. I verified data types to ensure they match the expected format (e.g., dates, numbers).

SELECT pg_typeof(unit_price) AS data_type
FROM analytics;


3. I validated the referential integrity of the data to ensure that relationships between tables are maintained by linking unique keys(primary keys, unique keys)

SELECT *
FROM products pr 
JOIN sales_report sr
ON pr.sku = sr.productsku
JOIN sales_by_sku sbs
on sr.productsku = sbs.productsku;