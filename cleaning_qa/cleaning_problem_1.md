# PROBLEM 01
Where it shows cost/pricing in this data, it needs to be divided by 1,000,000

## Solution
I created a User-Defined Function that divides a value by 1,000,000 and rounds it to 2 decimal points.

### Query
```sql
CREATE OR REPLACE FUNCTION dividebymil(num NUMERIC)
RETURNS NUMERIC AS
$$
BEGIN
    RETURN ROUND(num / 1000000, 2);
END;
$$ LANGUAGE plpgsql;
```

### QA
To ensure that I'm calculating the values properly using the UDF, I compared in-line calculations `ROUND(unit_price/1000000, 2)` vs the UDF `dividebymil(unit_price)` and it yields the exact result.

These are the queries I used:
#### Exploration findings
```sql
SELECT * FROM numbers_summary('analytics')
```
> This tells me that the MIN value in units_sold column is -89 which could be an indication that the transaction is a refund, so we don't want to count that towards revenue (`greater than 0`).

#### Main Query
```sql
WITH qa_analytics_revenue AS (
	SELECT
	units_sold,
	-- unit_price
	ROUND(unit_price/1000000, 2) as unit_price_v1,
	dividebymil(unit_price) as unit_price_v2,

	-- revenue
	ROUND(COALESCE(revenue, units_sold * unit_price)/1000000,2)
		as revenue_v1,
	dividebymil(COALESCE(revenue, units_sold * unit_price))
		as revenue_v2

	FROM	analytics
	WHERE	units_sold > 0
)
SELECT
	CASE
		WHEN unit_price_v1 != unit_price_v2 THEN 'no'
		ELSE 'y'
	END as unit_price_difference,
	CASE
		WHEN revenue_v1 != revenue_v2 THEN 'no'
		ELSE 'y'
	END as revenue_difference
FROM qa_analytics_revenue
ORDER BY
	unit_price_difference,
	revenue_difference
```