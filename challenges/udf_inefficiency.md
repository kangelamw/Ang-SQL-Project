## UDF Inefficiencies
I noticed an efficiency problem where running the UDF `overview_of_each_column('table_name')` on some tables could take an unnecessarily long time. I created a `MATERIALIZED VIEW` to solve this problem.

<br>

[This view](../tables_analysis/overview_of_each_column/_tables_general_overview.csv) aggregates the overviews of multiple tables into a single materialized view for easy reference and analysis.
  - Uses UNION ALL to combine the results from the overview_of_each_column function for each specified table.
  - Adds a table_name column to identify the source table for each row in the materialized view.

<br>

#### BEST USED ON:

Calling the function on analytics table:
  
`overview_of_each_column('analytics') `
- takes 31.362s

`SELECT * FROM tables_general_overview WHERE table_name = 'analytics' ORDER BY num_of_null_val DESC`
- takes 0.115s

<br>

#### The query creating this view is shown below:
```sql
CREATE MATERIALIZED VIEW tables_general_overview AS
  SELECT 'sales_by_sku' AS table_name, *
  FROM overview_of_each_column('sales_by_sku') 	

  UNION ALL
  SELECT 'sales_report' AS table_name, * 
  FROM overview_of_each_column('sales_report') 	

  UNION ALL
  SELECT 'products' AS table_name, * 
  FROM overview_of_each_column('products')	

  UNION ALL
  SELECT 'analytics' AS table_name, * 
  FROM overview_of_each_column('analytics')	

  UNION ALL
  SELECT 'all_sessions' AS table_name, * 
  FROM overview_of_each_column('all_sessions')
  ORDER BY	table_name,
        num_of_null_val DESC,
        num_of_distinct_val DESC;
```