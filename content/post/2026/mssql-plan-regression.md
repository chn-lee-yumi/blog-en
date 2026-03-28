---
title: "MSSQL Plan Regression Issue"
description: "Recently I encountered a classic performance issue in MSSQL called plan regression. In this post, I will share my experience of troubleshooting and solving this problem."
date: 2026-03-28T21:10:00+11:00
categories:
  - Learning
tags:
  - Database
---

## Background

Recently, I encountered a mysterious issue while writing SQL. Our test environment database is copied from the production environment every week. When I ran the SQL I wrote in the test environment, everything was fine, but when I deployed it to the production environment, it became extremely slow. What used to take around 200ms now takes 20s.

## Troubleshooting

Since the SQL I wrote is a long and complex one (actually a stored procedure), the first step was to identify which specific SQL statement was causing the slowdown. Using a binary search debugging method, I finally pinpointed this statement:

```sql
WITH Top6BaseDateTimeUTC AS (
    SELECT TOP 6 BaseDateTimeUTC
    FROM (
        SELECT BaseDateTimeUTC
        FROM GWForecasts WITH (NOLOCK)
        WHERE Product = 'WTHRBIT1HR'
        GROUP BY BaseDateTimeUTC
    ) t
    ORDER BY BaseDateTimeUTC DESC
)
SELECT DISTINCT
    f.Lat,
    f.Lon
INTO #ForecastGridWB
FROM GWForecasts f WITH (NOLOCK)
JOIN Top6BaseDateTimeUTC t
    ON f.BaseDateTimeUTC = t.BaseDateTimeUTC
WHERE f.Product = 'WTHRBIT1HR';
```

The statement is fetching data from the `GWForecasts` table, which contains tens of millions of rows. I tried changing `TOP 6` to `TOP 5` in the production environment, and the speed improved dramatically. However, in the test environment, `TOP 6` was still fast. Therefore, I suspected that there might be a threshold here, and exceeding it would lead to a performance regression.

So I tried increasing the `TOP 6` in the test environment, and finally, when I changed it to `TOP 11`, the performance regression issue occurred. So now I can reproduce the problem in the test environment, although the threshold is different.

Since the test environment database is synchronized weekly, it contains fewer data than the production environment. Therefore, it's reasonable that the threshold is different.

After asking AI, it informed me that this situation is very classic and is called plan regression. When SQL Server's query optimiser generates an execution plan, it may choose different execution plans to execute the query. When certain parameters or data distribution of the query change, the optimiser may choose a different execution plan, which may be less efficient, leading to performance regression.

I then analyzed the execution plan of this statement, which confirmed this point. Since the execution plan is in XML format, very long, and not suitable for human reading, I won't include it here.

I use Gemini to create a visualizer for execution plans, and you can also try it out: [SQL Server Query Plan Visualiser](https://chn-lee-yumi.github.io/SQL-Server-Query-Plan-Visualiser/)

## Solution

Solving this problem using a temporary table: (the solution was also provided by AI)

```sql
SELECT DISTINCT TOP 6 BaseDateTimeUTC
INTO #TopBaseDateTimeUTC
FROM GWForecasts WITH (NOLOCK)
WHERE Product = 'WTHRBIT1HR'
ORDER BY BaseDateTimeUTC DESC;

SELECT DISTINCT
    f.Lat,
    f.Lon
INTO #ForecastGridWB
FROM GWForecasts f WITH (NOLOCK)
JOIN #TopBaseDateTimeUTC t
    ON f.BaseDateTimeUTC = t.BaseDateTimeUTC
WHERE f.Product = 'WTHRBIT1HR';
```
