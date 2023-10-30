Question 1: Determine how many visitors visited a site more than once?

SQL Queries:

SELECT DISTINCT fullvisitorid, COUNT(*) AS count
FROM all_sessions
GROUP BY fullvisitorid
HAVING COUNT(*) > 1;

WITH ValueCounts AS (
  SELECT fullvisitorid, COUNT(*) AS count
  FROM all_sessions
  GROUP BY fullvisitorid
  HAVING COUNT(*) > 1
)
SELECT COUNT(*) AS unique_values_appearing_more_than_once
FROM ValueCounts;

Answer: 
The first query returns 794 rows and the second query returns the count which means 794 visitors out of 14,223 visitors in total visited more than once


Question 2: What is the average amount of time a visitor spends on a site?

SQL Queries:

WITH AvgTimeonsite AS (
  SELECT AVG(timeonsite) AS overall_avg_timeonsite
  FROM temp_all_sessions
  WHERE timeonsite > '0 minutes'::interval
)
SELECT DISTINCT tas.fullvisitorid, 
       CAST(AVG(tas.timeonsite) AS interval) AS avg_timeonsite,
       a.overall_avg_timeonsite AS overall_avg_timeonsite
FROM temp_all_sessions tas
JOIN AvgTimeonsite a ON 1=1
WHERE tas.timeonsite > '0 minutes'::interval
GROUP BY tas.fullvisitorid, a.overall_avg_timeonsite;


Answer:



Question 3: What were the two most used channel grouping by visitors in visiting sites?

SQL Queries:

SELECT channelgrouping, COUNT(*) AS count
FROM temp_all_sessions
GROUP BY channelgrouping
ORDER BY count DESC
LIMIT 2;

Answer:
From the query result, the most used channel groupings were Organic Search and Direct. An inference of this result could mean that most visitors did not visit sites through referrals, paid searches or affiliates.

"channelgrouping"	"count"
"Organic Search"	8653
"Direct"		2997
"Referral"		2580
"Paid Search"		509
"Affiliates"		265
"Display"		125
"(Other)"		5


Question 4: 

SQL Queries:

Answer:



Question 5: 

SQL Queries:

Answer:
