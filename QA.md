## What are your risk areas? Identify and describe them.
**<u>Significant amounts of missing or null values in critical columns</u>**

- These gaps can lead to incomplete insights and may skew results if not properly addressed through data cleaning or imputation methods.
  > For revenue, `COALESCE` was used to fill in NULL values with `unit_cost * units_sold`

**<u>Inconsistencies in Data Types and formats</u>**
- These were mainly on time-related values. It tells us how long the visitors spent on the site, or when their session started. I had to verify what the values could be and put in additional columns showing these proper values.
  > For instance, timeonsite is in **seconds**, time is in **milliseconds**, and visitstarttime is a lot of numbers, but it's a **date** with **time**.

<br>

## QA Process
Describe your QA process and include the SQL queries used to execute it.
> Read more about the my Data Exploration Process [here.](./tables_analysis/data_exploration.md)

### 1. I started with identifying the datatypes in each column and the entries in the table.
**Queries:**
```
SELECT * FROM all_sessions
SELECT * FROM analytics
SELECT * FROM sales_report
SELECT * FROM sales_by_sku
SELECT * FROM products
```

<br>

### 2. This is how I checked for total number of rows, null and distinct values.

**Queries:**
```
SELECT * FROM tables_general_overview WHERE table_name = 'all_sessions'
	ORDER BY num_of_null_val DESC;

SELECT * FROM tables_general_overview WHERE table_name = 'analytics'
  ORDER BY num_of_null_val DESC;

SELECT * FROM tables_general_overview WHERE table_name = 'products'
  ORDER BY num_of_null_val DESC;

SELECT * FROM tables_general_overview WHERE table_name = 'sales_by_sku'
  ORDER BY num_of_null_val DESC;

SELECT * FROM tables_general_overview WHERE table_name = 'sales_report'
  ORDER BY num_of_null_val DESC;
```
- *What is `tables_general_overview('tablename')`?* A UDF.
	- Click [here](./tables_analysis/udf_overview_of_each_column.md) for the longer answer.

<br>

### 3. I used `numbers_summary('tablename')` to compute the Five Number Summary of each numeric column in a table.
**Queries:**
```
SELECT * FROM numbers_summary('all_sessions')
SELECT * FROM numbers_summary('analytics')
SELECT * FROM numbers_summary('products')
SELECT * FROM numbers_summary('sales_report')
SELECT * FROM numbers_summary('sales_by_sku')
```
- *What is `numbers_summary('tablename')`?* A UDF.
	- Click [here](./tables_analysis/udf_numbers_summary.md) for the longer answer.

<br>

## For the UDFs used
Prior to using the following UDFs to answer the questions, I've ensured that each function was producing the same output as doing manual computations.

It slowed me down in the beginning where I was crafting and debugging the UDFs, but it saved me a lot of time as these are calculations I would have had to keep referring back to as I went through the data.

View full documentation for each function below:
- [`overview_of_each_column('tablename')`](./tables_analysis/udf_overview_of_each_column.md)
- [`numbers_summary('tablename'`)](./tables_analysis/udf_numbers_summary.md)
- [`dividebymil(columnname)`](./tools_etc/udf_dividebymil.md)