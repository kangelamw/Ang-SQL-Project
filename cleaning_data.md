# What issues will you address by cleaning the data?

**Problem 01:**
Where it shows cost/pricing in this data, it needs to be divided by 1,000,000
> Solution: I created a User-Defined Function: [dividebymil](./tools_etc/udf_dividebymil.md) that divides a value by 1,000,000 and rounds it to 2 decimal points

**Problem 02:**
Some values in the revenue table is empty, but I saw that there are no null values in the unit_price table. I've also observed that units_sold is greater than 1, indicating that there was a sale.
> Solution: 
I created a materialized view of table containing a list of all the unique_fullvisitorids to have ever appeared in our system.

**Problem 03:**
I need a table to refer back to when ensuring I am calculating values that should be unique to a visitor.
> Solution: I need a table to refer back to when ensuring I am calculating values that should be unique to a visitor.

**Problem 04:**
Inconsistency in the following time-related entries:
- `analytics.timeonsite` is in __seconds__
- `analytics.visitstarttime` is in __seconds__
- `all_sessions.timeonsite` is in __seconds__
- `all_sessions.time` is in __milliseconds__
> Solution: I added new columns with updated and properly formatted values.

# Queries:
Below, provide the SQL queries you used to clean your data.

## **Problem 01**
Where it shows cost/pricing in this data, it needs to be divided by 1,000,000

### **Solution 01**

I created a User-Defined Function that divides a value by 1,000,000 and rounds it to 2 decimal points
```
CREATE OR REPLACE FUNCTION dividebymil(num NUMERIC)
RETURNS NUMERIC AS
$$
BEGIN
    RETURN ROUND(num / 1000000, 2);
END;
$$ LANGUAGE plpgsql;
```
> [QA on this query for cleaning problem 1](./cleaning_qa/cleaning_problem_1.md)

## **Problem 02**
Some values in the revenue table is empty, but I saw that there are no null values in the unit_price table. I've also observed that units_sold is greater than 1, indicating that there was a sale.

### **Solution 02**
I created a materialized view of table containing a list of all the unique_fullvisitorids to have ever appeared in our system.

This table shows the full visitor id, sale date, number of units sold, unit price and revenue.

>If revenue is null, it calculates `units_sold * unit_price`. I used `COALESCE` and the `dividebymil` UDF to divide the relevant numbers by 1M and round to to decimal places.

This view utilizes the [`UDF: dividebymil(num)`](./tools_etc/udf_dividebymil.md) to provide proper pricing.
```
CREATE MATERIALIZED VIEW  analytics_revenue_table AS
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
> [QA on this query for cleaning problem 2](./cleaning_qa/cleaning_problem_2.md)

## **Problem 03**
I need a table to refer back to when ensuring I am calculating values that should be unique to a visitor.

### **Solution 03**

I created a materialized view of table containing a list of all the unique_fullvisitorids to have ever appeared in our system.
> [Full table](./materialized_views/mat_unique_fullvisitorids.csv)
```
CREATE MATERIALIZED VIEW unique_fullvisitorids AS
	SELECT DISTINCT all_sessions.fullvisitorid
  	FROM all_sessions
	UNION
 	SELECT DISTINCT analytics.fullvisitorid
  	FROM analytics;
```

## **Problem 04**
Inconsistency in the ff. time-related entries:

- `analytics.timeonsite` is in __seconds__
- `analytics.visitstarttime` is in __seconds__
- `all_sessions.timeonsite` is in __seconds__
- `all_sessions.time` is in __milliseconds__
### **Solution 04**

I added new columns with updated and properly formatted values.

- `all_sessions` table
```
-- Adding new columns
	ALTER TABLE all_sessions
	ADD COLUMN timeonsite_proper INTERVAL,
	ADD COLUMN time_proper TIME;

-- Updating new columns
	-- Update timeonsite_proper
	UPDATE analytics
	SET timeonsite_proper = timeonsite * INTERVAL '1 second';
		
	-- Update visitstarttime_proper
	UPDATE analytics
	SET visitstarttime_proper = TO_TIMESTAMP(visitstarttime);
```

- `analytics` table
```
-- Adding new columns
	ALTER TABLE analytics
	ADD COLUMN timeonsite_proper INTERVAL,
	ADD COLUMN visitstarttime_proper TIMESTAMP;

-- Updating new columns
	-- Update timeonsite_proper
	UPDATE all_sessions
	SET timeonsite_proper = timeonsite * INTERVAL '1 second';
			
	-- Update time_proper
	UPDATE all_sessions
	SET time_proper = time * INTERVAL '1 millisecond';
```

> [QA on this query for cleaning problem 4](./cleaning_qa/cleaning_problem_4.md)