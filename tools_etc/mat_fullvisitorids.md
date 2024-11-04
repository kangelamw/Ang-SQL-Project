 # UNIQUE FULL VISITOR IDS

A list of all unique/`DISTINCT fullvisitorids` FROM `analytics` and `all_sessions` table.

> [Full csv](../materialized_views/mat_unique_fullvisitorids.csv)

### Query:
```
CREATE MATERIALIZED VIEW unique_fullvisitorids AS
  SELECT DISTINCT all_sessions.fullvisitorid
    FROM all_sessions
  UNION
  SELECT DISTINCT analytics.fullvisitorid
    FROM analytics;
```