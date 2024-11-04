## Find the total number of unique visitors by referring sites


### Answer:
![referral source](../img/referral.png)
```
WITH referredVisits AS (
  SELECT
    als.fullvisitorid,
    als.pagepathlevel1 AS referralSource,
    ROW_NUMBER() OVER 	(PARTITION BY als.fullvisitorid
                        ORDER BY als.date, als.date)
                        AS referralNumber
    FROM	all_sessions als
    WHERE	als.channelgrouping = 'Referral'
)
  SELECT	rVs.referralSource,
        COUNT(DISTINCT rVs.fullvisitorid) AS uniqueVisitorCount
  FROM	referredVisits rVs
  WHERE	rVs.referralNumber = 1
  GROUP BY	rVs.referralSource
  ORDER BY	uniqueVisitorCount DESC;
```

### QA:
This query counts the number of DISTINCT fullVisitor IDs that was referred to the site

```
SELECT COUNT(DISTINCT als.fullvisitorid)
FROM all_sessions als
WHERE als.channelgrouping = 'Referral'; -- It returns 2419
```

I'm using a CTE to SUM the counts of unique fullVisitorIds from the resulting query to verify that we ended with the same count:
```
WITH QA_table AS (
	WITH referredVisits AS (
		SELECT
			als.fullvisitorid,
			als.pagepathlevel1 AS referralSource,
				ROW_NUMBER() OVER 	(PARTITION BY als.fullvisitorid
									 ORDER BY als.date, als.date) AS referralNumber
		FROM	all_sessions als
		WHERE	als.channelgrouping = 'Referral'
	)
	SELECT	rVs.referralSource,
			COUNT(DISTINCT rVs.fullvisitorid) AS uniqueVisitorCount
	FROM	referredVisits rVs
	WHERE	rVs.referralNumber = 1
	GROUP BY	rVs.referralSource
	ORDER BY	uniqueVisitorCount DESC
)	
```

```
SELECT SUM(uniqueVisitorCount) FROM QA_table -- Returns 2419!
```