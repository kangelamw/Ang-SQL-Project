# Data Exploration: An overview:
We have the ff. tables:
  * all_sessions
  * analytics
  * products
  * sales_report
  * sales_by_sku

## PROCESS 1
Identifying what the columns look like and the data in them.
```
SELECT * FROM all_sessions
SELECT * FROM analytics
SELECT * FROM sales_report
SELECT * FROM sales_by_sku
SELECT * FROM products
```

## PROCESS 2
I created a User-Defined-Function to get a generalized overview of the columns in each table to know:
  - Which columns in a table needs reviewed for duplicates
  - If I can extrapolate some data or perform calculations on some columns to fill in null values where relevant

*Sometimes, the count of distinct values is irrelevant to the column*
*(f.e. all_sessions.itemQuantity -- It doesn't matter if multiple items are different quantities)*

The query is shown below. [Click here to view documentation](./tables_analysis/udf_overview_of_each_column.md)
```
CREATE OR REPLACE FUNCTION overview_of_each_column(tablename TEXT)
RETURNS TABLE(
    column_name TEXT,
    total_num_of_rows BIGINT,
    num_of_null_val BIGINT,
    num_of_distinct_val BIGINT
) AS $$
DECLARE
    col RECORD;
    sql TEXT;
BEGIN
    FOR col IN
        SELECT c.table_schema, c.column_name
        FROM information_schema.columns AS c
        WHERE c.table_name = tablename
    LOOP
        sql := format('
            SELECT %L,         -- Column name as a literal string
                   COUNT(*),   -- Count of rows
                   COUNT(*) - COUNT(%I),  -- Count of NULL values
                   COUNT(DISTINCT %I)     -- Count of distinct non-NULL values
            FROM %I.%I',       -- Schema-qualified table name
            col.column_name,   -- Literal string for the column name
            col.column_name,   -- Column identifier for count of NULLs
            col.column_name,   -- Column identifier for count of distinct non-NULLs
            col.table_schema,  -- Schema name
            tablename);        -- Table name

        RETURN QUERY EXECUTE sql;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

### Getting a general overview of each table using UDF: overview_of_each_column('table_name')
General overview of columns in the table: **[all_sessions](./tables_analysis/overview_of_each_column/quick_all_sessions.csv)**
```
  -- Query 1: UDF: overview_of_each_column('all_sessions')
    SELECT * FROM overview_of_each_column('all_sessions')
      ORDER BY num_of_null_val DESC; -- 0.368s to run
  /* OR */
  -- Query 2:
    SELECT * FROM tables_general_overview WHERE table_name = 'all_sessions'
      ORDER BY num_of_null_val DESC; -- 0.120s to run
```

General overview of columns in the table: **[analytics](./tables_analysis/overview_of_each_column/quick_analytics.csv)**
```
  -- Query 1: UDF: overview_of_each_column('analytics')
    SELECT * FROM overview_of_each_column('analytics')
      ORDER BY num_of_null_val DESC; -- 39.563s to run
  /* OR */
  -- Query 2:
    SELECT * FROM tables_general_overview WHERE table_name = 'analytics'
      ORDER BY num_of_null_val DESC; -- 0.115s to run
```
General overview of columns in the table: **[products](./tables_analysis/overview_of_each_column/quick_products.csv)**
```
  -- Query 1: UDF: overview_of_each_column('products')
    SELECT * FROM overview_of_each_column('products')
      ORDER BY num_of_null_val DESC; -- 1.832s to run
  /* OR */
  -- Query 2:
    SELECT * FROM tables_general_overview WHERE table_name = 'products'
      ORDER BY num_of_null_val DESC; -- 0.089s to run
```

General overview of columns in the table: **[sales_report](./tables_analysis/overview_of_each_column/quick_sales_report.csv)**
```
  -- Query 1: UDF: overview_of_each_column('sales_report')
    SELECT * FROM overview_of_each_column('sales_report')
      ORDER BY num_of_null_val DESC; -- 1.828s to run
  /* OR */
  -- Query 2:
    SELECT * FROM tables_general_overview WHERE table_name = 'sales_report'
      ORDER BY num_of_null_val DESC; -- 0.101s to run
```
General overview of columns in the table: **[sales_by_sku](./tables_analysis/overview_of_each_column/quick_sales_by_sku.csv)**
```
  -- Query 1: UDF: overview_of_each_column('sales_by_sku')
    SELECT * FROM overview_of_each_column('sales_by_sku')
      ORDER BY num_of_null_val DESC; -- 1.676s to run
  /* OR */
  -- Query 2:
    SELECT * FROM tables_general_overview WHERE table_name = 'sales_by_sku'
      ORDER BY num_of_null_val DESC; -- 0.089s to run
```

## PROCESS 3
I created a User-Defined-Function to get a generalized overview of the numbers within the columns of each table. This function finds Numeric Data types in the table and performs a `Five Number Summary` calculation on each.


The query is shown below. [Click here to view documentation](./tables_analysis/udf_numbers_summary.md)
```
CREATE OR REPLACE FUNCTION numbers_summary(p_tableName TEXT)
RETURNS TABLE (
    columnName TEXT,
    minVal NUMERIC,
    maxVal NUMERIC,
    avgVal NUMERIC,
    quartile1 NUMERIC,
    quartile2 NUMERIC,
    quartile3 NUMERIC,
    interquartileRange NUMERIC
) AS $$
DECLARE
    col RECORD;
    sql TEXT;
BEGIN
    -- Loop through each numeric column in the specified table
    FOR col IN
        SELECT column_name
        FROM information_schema.columns
        WHERE table_name = lower(p_tableName)
          AND table_schema = 'public' -- Specifying the schema to avoid ambiguity
          AND data_type IN ('integer', 'numeric', 'real', 'double precision', 'smallint', 'bigint', 'decimal')
    LOOP
        -- Build the SQL query for the current column
        sql := format(
            'SELECT 
                %1$L AS columnName,
                MIN(%2$I)::NUMERIC AS minVal,
                MAX(%2$I)::NUMERIC AS maxVal,
                ROUND(AVG(%2$I),2)::NUMERIC AS avgVal,
                percentile_cont(0.25) WITHIN GROUP (ORDER BY %2$I)::NUMERIC AS quartile1,
                percentile_cont(0.5) WITHIN GROUP (ORDER BY %2$I)::NUMERIC AS quartile2,
                percentile_cont(0.75) WITHIN GROUP (ORDER BY %2$I)::NUMERIC AS quartile3,
                (percentile_cont(0.75) WITHIN GROUP (ORDER BY %2$I) - percentile_cont(0.25) WITHIN GROUP (ORDER BY %2$I))::NUMERIC AS interquartileRange
             FROM %3$I.%4$I
             WHERE %2$I <> 0 AND %2$I IS NOT NULL',
            col.column_name,    -- %1$: columnName as a literal
            col.column_name,    -- %2$: columnName as an identifier (used multiple times)
            'public',           -- %3$: Schema name
            lower(p_tableName)  -- %4$: Table name in lowercase
        );
        RETURN QUERY EXECUTE sql;
    END LOOP;
END;
```

### Getting a general overview of the numbers in each table using UDF: numbers_summary('table_name')
General overview of columns in the table: **[all_sessions](./tables_analysis/numbers_summary/numsum_all_sessions.csv)**
```
    SELECT * FROM numbers_summary('all_sessions')
```

General overview of columns in the table: **[analytics](./tables_analysis/numbers_summary/numsum_analytics.csv)**
```
    SELECT * FROM numbers_summary('analytics')
```
General overview of columns in the table: **[products](./tables_analysis/numbers_summary/numsum_products.csv)**
```
    SELECT * FROM numbers_summary('products')
```

General overview of columns in the table: **[sales_report](./tables_analysis/numbers_summary/numsum_sales_report.csv)**
```
    SELECT * FROM numbers_summary('sales_report')
```
General overview of columns in the table: **[sales_by_sku](./tables_analysis/numbers_summary/numsum_sales_by_sku.csv)**
```
    SELECT * FROM numbers_summary('sales_by_sku')
```