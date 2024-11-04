# Question 2: What are the site's peak hours per country?
This aims to determine peak hours for site visits across different cities and countries to help us optimize our server space allocation and plan site updates more efficiently while ensuring we have 100% up time at peak hours.

### Answer:
This tells us the peak hours in Canada and the US. From here, we can find optimal update windows and when to allocate more server space per country, per hour. *Also for cutting down on server costs*

Example use case:
![alt text](../img/dataq2.png)

## QA
Ensuring we're only counting 24 hours.
```
WITH
visitor_times AS (
    -- Use visitstarttime_proper for already converted times
    SELECT
        a.fullVisitorId,
        als.city,
        als.country,
        visitstarttime_proper,
        TO_CHAR(visitstarttime_proper, 'HH24:00') AS visit_hour
    FROM
        analytics a
    JOIN
        unique_fullvisitorids u ON a.fullVisitorId = u.fullVisitorId
    JOIN
        all_sessions als ON a.fullVisitorId = als.fullVisitorId
    WHERE
        visitstarttime_proper IS NOT NULL
        AND als.city IS NOT NULL
        AND als.country IS NOT NULL
        AND als.country != '(not set)'
)
SELECT DISTINCT visit_hour
FROM visitor_times
ORDER BY visit_hour;
```

Finding total visit counts per country:
```
WITH
visitor_times AS (
    -- Use visitstarttime_proper for already converted times
    SELECT
        a.fullVisitorId,
        als.city,
        als.country,
        visitstarttime_proper,
        TO_CHAR(visitstarttime_proper, 'HH24:00') AS visit_hour
    FROM
        analytics a
    JOIN
        unique_fullvisitorids u ON a.fullVisitorId = u.fullVisitorId
    JOIN
        all_sessions als ON a.fullVisitorId = als.fullVisitorId
    WHERE
        visitstarttime_proper IS NOT NULL
        AND als.city IS NOT NULL
        AND als.country IS NOT NULL
        AND als.country != '(not set)'
)
SELECT country, COUNT(*) AS num_records
FROM visitor_times
GROUP BY country
ORDER BY num_records DESC;
```
> Majority of our visitors are coming from the US!


**SQL Query:**
```
WITH
visitor_times AS (
    -- Use visitstarttime_proper for already converted times
    SELECT
        a.fullVisitorId,
        als.city,
        als.country,
        visitstarttime_proper,
        TO_CHAR(visitstarttime_proper, 'HH24:00') AS visit_hour
    FROM
        analytics a
    JOIN
        unique_fullvisitorids u ON a.fullVisitorId = u.fullVisitorId
    JOIN
        all_sessions als ON a.fullVisitorId = als.fullVisitorId
    WHERE
        visitstarttime_proper IS NOT NULL
        AND als.city IS NOT NULL
        AND als.country IS NOT NULL
        AND als.country != '(not set)'
), visit_counts AS (
    -- Query to count the number of visits per hour per city and country
    SELECT
        country,
        city,
        visit_hour,
        COUNT(*) AS num_visits
    FROM
        visitor_times
    GROUP BY
        country,
        city,
        visit_hour
    ORDER BY
        num_visits DESC,
        country,
        city,
        visit_hour
), time_of_day_visitors AS (
    -- Avg num of visits per hour, per country
    SELECT 
        country,
        city,
        visit_hour,
        ROUND(AVG(num_visits)) AS visitor_avg
    FROM 
        visit_counts
    GROUP BY
        country,
        city,
        visit_hour
    ORDER BY
        visitor_avg DESC,
        visit_hour,
        country,
        city
), mode_visit_time AS (
    -- Find the mode (most popular visit hour) across countries
    SELECT
        country,
        visit_hour,
        SUM(visitor_avg) AS total_visits,
        RANK() OVER (
            PARTITION BY country
            ORDER BY SUM(visitor_avg) DESC
        ) AS visit_rank
    FROM
        time_of_day_visitors
    GROUP BY
        country,
        visit_hour
    ORDER BY
        total_visits DESC
)
-- Select the top 3 most popular visit times for each country to show in a summary table
SELECT
    country,
    MAX(CASE WHEN visit_rank = 1 THEN visit_hour END) AS top_1_hour,
    MAX(CASE WHEN visit_rank = 1 THEN total_visits END) AS top_1_visits,
    MAX(CASE WHEN visit_rank = 2 THEN visit_hour END) AS top_2_hour,
    MAX(CASE WHEN visit_rank = 2 THEN total_visits END) AS top_2_visits,
    MAX(CASE WHEN visit_rank = 3 THEN visit_hour END) AS top_3_hour,
    MAX(CASE WHEN visit_rank = 3 THEN total_visits END) AS top_3_visits
FROM
    mode_visit_time
WHERE
    visit_rank <= 3
GROUP BY
    country
ORDER BY
    country;

--- convert the previous query into another CTE to look for specific countries:
SELECT * FROM select_country WHERE country IN ('Canada', 'United States')
```