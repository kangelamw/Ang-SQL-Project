 # ANALYTICS REVENUE TABLE

 It is a materialized view I created from using `dividebymil` on `unit_price` and `revenue`.
 
 It shows the `fullvisitorid`, the number of units_sold to them, and the session `date`.
 
 It uses `COALESCE` to handle cases where revenue is null by calculating `units_sold * unit_price`.

> [Full csv](../materialized_views/mat_analytics_revenue_table.csv)

### Query:
```sql
CREATE MATERIALIZED VIEW analytics_revenue_table AS
  SELECT fullvisitorid,
    date,
    units_sold,
    dividebymil(unit_price) AS unit_price,
    COALESCE(dividebymil(revenue), units_sold::numeric * dividebymil(unit_price)) AS revenue
   FROM analytics
  WHERE units_sold > 0
  GROUP BY fullvisitorid, date, units_sold, unit_price, revenue
  ORDER BY (COALESCE(dividebymil(revenue), units_sold::numeric * dividebymil(unit_price))) DESC;
```