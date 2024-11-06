# PROBLEM 02
Some values in the revenue table is empty, but I saw that there are no null values in the unit_price table. I've also observed that units_sold is greater than 0, indicating that there was a sale.

## Solution

I created a materialized view of table containing a list of all the `unique_fullvisitorids` to have ever appeared in our system.

This table shows the full visitor id, sale date, number of units sold, unit price and revenue.

>If revenue is null, it calculates `units_sold * unit_price`. I used `COALESCE` and the `dividebymil` UDF to divide the relevant numbers by 1M and round to 2 decimal places.

This view utilizes the [`UDF: dividebymil(num)`](../tools_etc/udf_dividebymil.md) to provide proper pricing.

### Query
```sql
CREATE MATERIALIZED VIEW analytics_revenue_table AS
	SELECT
	    fullvisitorid,
	    date,
	    units_sold,
	    dividebymil(unit_price) AS unit_price,
	    COALESCE(dividebymil(revenue),
	        	 units_sold * dividebymil(unit_price)
	    		 ) AS revenue
	FROM
	    analytics
	WHERE
	    units_sold > 0
	ORDER BY
	    revenue DESC
```

### QA
I compared the extrapolated values from the original values and decided that the minute differences between the two is an acceptable margin of error.
> The difference between the two values ranges `between $0.05 and $9.00` which is `0.03% to 6.01%`
```sql
WITH
differences_table AS (
	SELECT
		units_sold,
		dividebymil(unit_price) as unit_price,
		dividebymil(revenue) as revenue_v1,
		units_sold * dividebymil(unit_price) as revenue_v2,
		dividebymil(revenue) - units_sold * dividebymil(unit_price) as difference
	FROM analytics
	WHERE
		units_sold > 0 AND revenue IS NOT NULL
	ORDER BY unit_price DESC	
),
percent_diff AS (
	SELECT
	revenue_v1,
	revenue_v2,
	difference,
	ROUND((difference / AVG(revenue_v1 + revenue_v2) OVER ()) * 100, 2) AS percent_diff
FROM differences_table
)

SELECT
	MIN(dt.difference) as minDiff,
	MAX(dt.difference) as maxDiff,
	MIN(pd.percent_diff) as percent_min,
	MAX(pd.percent_diff) as percent_max
FROM differences_table dt
CROSS JOIN percent_diff pd
```