Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
WITH MaxTransactionRevenues AS (
    SELECT tas.country, tas.city, ta.unit_price * tp.orderedquantity AS transaction_revenue
    FROM temp_all_sessions tas
    JOIN temp_analytics ta ON tas.visitid = ta.visitid
    JOIN temp_products tp ON tas.productsku = tp.productsku
)
SELECT country, city, MAX(transaction_revenue) AS max_transactionrevenues
FROM MaxTransactionRevenues
WHERE transaction_revenue = (
    SELECT MAX(transaction_revenue)
    FROM MaxTransactionRevenues
)
GROUP BY country, city;


Answer:
United States was the country with the highest level of transaction revenues on the site, while New York and Chicago were the two cities (both located in the United States) with the highest level of transactions on the site.



**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
SELECT tas.city, tas.country, AVG(CASE WHEN tp.orderedquantity > 0 THEN tp.orderedquantity END) AS avg_orderedquantity
FROM temp_all_sessions tas
JOIN temp_products tp ON tas.productsku = tp.productsku
JOIN temp_analytics ta ON tas.visitid = ta.visitid
GROUP BY tas.city, tas.country
HAVING AVG(CASE WHEN tp.orderedquantity > 0 THEN tp.orderedquantity END) IS NOT NULL
ORDER BY tas.city, tas.country DESC;

Or the syntax below could be used to find the average number of ordered products in each city specifically:
SELECT DISTINCT tas.city, AVG(tp.orderedquantity) 
		OVER (PARTITION BY tas.city) AS avg_orderedquantity
FROM temp_all_sessions tas
JOIN temp_products tp 
	ON tas.productsku = tp.productsku
JOIN temp_analytics ta
	ON tas.visitid = ta.visitid
WHERE tp.orderedquantity > 0
ORDER BY tas.city;

And to get the average per country specifically, we simply swap out city for country in the syntax above.

Answer:
The result set shows a varying range for the average number of products ordered in each city and country



**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

SELECT DISTINCT tas.city, tas.country, tas.v2productcategory,  MAX(tp.orderedquantity)
		OVER (PARTITION BY tas.v2productcategory) AS orderedqtyproduct
FROM temp_all_sessions tas
JOIN temp_products tp 
	ON tas.productsku = tp.productsku
WHERE tp.orderedquantity != 0
ORDER BY orderedqtyproduct;

Answer:
Yes, from the result set, we can see that visitors order most form the Home products category which varies widely from Accessories, Apparel, Electronics etc.




**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

WITH TopSellingProducts AS (
  SELECT tas.country, tas.city, tp.name AS top_selling_product, tp.orderedquantity,
    ROW_NUMBER() OVER(PARTITION BY tas.city, tas.country
      ORDER BY tp.orderedquantity DESC) AS product_rank
  FROM temp_all_sessions tas
  JOIN temp_products tp ON tas.productsku = tp.productsku
  GROUP BY tas.city, tas.country, tp.name, tp.orderedquantity
)
SELECT country, city, top_selling_product AS top_selling_product_ranked
FROM TopSellingProducts
WHERE product_rank = 1;



Answer:
The top-selling products varied greatly across each city/country. I did note that a pattern that all products were items for personal use. 




**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
SELECT DISTINCT als.city, als.country, SUM (als.totaltransactionrevenue) 
		OVER (PARTITION BY als.city, als.country) AS revenue_generated
FROM all_sessions als
JOIN temp_products tp 
	ON als.productsku = tp.productsku
JOIN temp_analytics ta
	ON als.visitid = ta.visitid
WHERE totaltransactionrevenue != 0
ORDER BY revenue_generated;

SELECT DISTINCT als.country, SUM (als.totaltransactionrevenue) 
		OVER (PARTITION BY als.country) AS revenue_generated
FROM all_sessions als
JOIN temp_products tp 
	ON als.productsku = tp.productsku
JOIN temp_analytics ta
	ON als.visitid = ta.visitid
WHERE totaltransactionrevenue != 0
ORDER BY revenue_generated DESC;


Answer:
From the result set of the queries, we can deduce that the United States was the country with the most revenue generated as also having the cities with the highest revenue generated compared to other countries.

The impact of this analysis can be used to evaluate the performance of marketing campaigns, product launches, or other initiatives in specific regions. It can help assess whether investments in certain areas yield the expected returns.

Additional analysis can be performed to delve deeper into the factors influencing revenue in different regions, this might include customer demographics, market conditions, or competitive landscape.

High-revenue cities or countries may be considered as potential areas for geographical expansion or investment, while low-revenue regions may require additional attention or adjustments to strategies.

In summary, the provided result set offers insights into the revenue generated by city and country, allowing organizations to make data-driven decisions, track performance, and optimize strategies for different regions.
The analysis can aid in understanding revenue patterns and their implications for the business.












