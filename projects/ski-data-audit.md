---
# ---- The card on the Projects page ----
title: Auditing a Colorado Ski Resort Dataset
tag: SQL · Data Quality Audit
card-summary: Testing whether a season of ski resort data can be trusted, then cleaning it with SQL.
icon: paper
color: sage

# ---- The top of the project page ----
eyebrow: INFO 3401 · Solo
summary: Can we trust this data to answer which Colorado resorts drew the most visitors in the 2025–2026 ski season?

meta: Role | Solo project
meta: Methods | Data quality audit, data cleaning
meta: Tools | SQL (SQLite)

# ---- The left side: audit scorecard + raw vs. cleaned ranking ----
# (the visual itself lives in files/ski-data-audit.html)
embed: files/ski-data-audit.html
---

## Overview

This past summer, I interned for an audit firm where I spent most of my work reviewing internal IT controls, checking to see if those controls were legitimate, accurate, and functioning as anticipated. I wanted to apply some of those same practices to this project; run an "audit" of a dataset to understand if the data is fit to use for analysis. I used the "CO_Ski_Database", containing 249 rows, that tracks weekly status, snowfall, ticket price, and number of visitors (season pass, multi day, and single day ticket holders) for 12 ski resorts across Colorado. I thought this would be a fun dataset to work with, given my home state is Colorado, and I am an annual Icon pass holder.

## Method

I brainstormed with Claude AI to come up with my methodology to audit this data, settling on testing for:

1. **Completeness:** are there gaps in the data? If so, how big are they?
2. **Validity:** are the values in a usable, acceptable, standard form?
3. **Uniqueness:** are there repeats?
4. **Consistency:** do values that mean the same thing get recorded the same way?
5. **Reasonableness:** does the data make sense in the real world?

I framed this project through one central question: *Can we trust this data to answer which Colorado resorts drew the most visitors in the 2025-2026 ski season?*

## 1. Completeness

To test for any missing values, I checked every column in the Ski_Fields table for true NULL's, empty text, and the word "NULL". I found that Snowfall is the only incomplete column; 14 of 249 rows have no snowfall recorded, and every other column is complete. I then grouped the Snowfall gaps by resort, to understand if the gaps were contained to one resort. I found the missing data is spread across most resorts, then counted each week where snowfall is missing. This showed me that no resort misses reporting on Snowfall for more than 3 weeks, suggesting the gaps are occasional mistakes. The completeness check passes in regards to the central question.

<details>
<summary>SQL for this check</summary>

```sql
-- Find any row with a missing value in any column
SELECT
*
FROM
Ski_Fields
WHERE
RecordID IS NULL OR RecordID = '' OR RecordID = 'NULL'
OR Resort IS NULL OR Resort = '' OR Resort = 'NULL'
OR Location IS NULL OR Location = '' OR Location = 'NULL'
OR WeekStart IS NULL OR WeekStart = '' OR WeekStart = 'NULL'
OR Status IS NULL OR Status = '' OR Status = 'NULL'
OR Snowfall IS NULL OR Snowfall = '' OR Snowfall = 'NULL'
OR Ticket_Price IS NULL OR Ticket_Price = '' OR Ticket_Price = 'NULL'
OR Season_Pass_Visitors IS NULL OR Season_Pass_Visitors = '' OR Season_Pass_Visitors = 'NULL'
OR Multi_Day_Visitors IS NULL OR Multi_Day_Visitors = '' OR Multi_Day_Visitors = 'NULL'
OR Day_Ticket_Visitors IS NULL OR Day_Ticket_Visitors = '' OR Day_Ticket_Visitors = 'NULL'
```

```sql
-- Where are the snowfall gaps?
SELECT
Resort,
COUNT (*) as Missing_Weeks
FROM
Ski_Fields
WHERE
Snowfall IS NULL OR Snowfall = '' OR Snowfall = 'NULL'
GROUP BY
Resort
```

</details>

## 2. Validity

To test if the values are in a usable and acceptable form, I checked every column in the Ski_Fields table. My checks were dependent on the type of variable. I found that 249 rows in the RecordID column share 240 numbers, so some ID's appear more than once. As well, both the Snowfall and Ticket_Price columns are stored in text, so, while they look like whole numbers, they cannot be used as legitimate integers to compare or average prices or snowfall. I used a Min/Max check on both variables to prove this; both failed to give accurate minimum or maximum values. So, 8 of 10 columns are valid to answer the central question.

| Column | What it should look like | Result | Comments |
|---|---|---|---|
| RecordID | Whole number in a reasonable range | Pass | All IDs are between 1001 and 1240, which is reasonable, and both are whole numbers. There are 249 rows, but only 240 IDs. |
| Resort | Only real resort names | Pass | All resorts are legitimate, capitalization varies. |
| Location | Always in CO | Pass | All locations are in Colorado. |
| WeekStart | A real date within the ski season | Pass | All dates are within November and April, in the ski season. |
| Status | Only "Open" or "Closed" | Pass | Only open or closed are options. |
| Snowfall | Reasonable amounts of snowfall: no negatives and no insanely high numbers | Fail | 14 weeks are blank, all weeks have an appropriate amount of snowfall, but stored as text, so unsuitable for analysis. |
| Ticket_Price | Should have a $ sign and be a whole number | Fail | All have $ and are whole numbers, but stored as text, so unsuitable for analysis. |
| Season_Pass_Visitors | Not negative | Pass | No negative visitors. |
| Multi_Day_Visitors | Not negative | Pass | No negative visitors. |
| Day_Ticket_Visitors | Not negative | Pass | No negative visitors. |

<details>
<summary>SQL for this check</summary>

```sql
-- RecordID
SELECT
  MIN(RecordID) AS Lowest_ID,
  MAX(RecordID) AS Highest_ID,
  COUNT(*) AS Total_Rows
FROM
  Ski_Fields
```

```sql
-- Resort
SELECT DISTINCT
Resort
FROM
Ski_Fields
```

```sql
-- Location
SELECT DISTINCT
Location
FROM
Ski_Fields
```

```sql
-- WeekStart
SELECT
MIN(WeekStart) AS "FirstWeek",
MAX(WeekStart) AS "LastWeek"
FROM
Ski_Fields
```

```sql
-- Status
SELECT DISTINCT
Status
FROM
Ski_Fields
```

```sql
-- Snowfall
SELECT
Snowfall,
COUNT(*) AS Weeks
FROM
Ski_Fields
GROUP BY
Snowfall
```

```sql
-- Ticket_Price
SELECT DISTINCT
Ticket_Price
FROM
Ski_Fields

SELECT
MAX (Ticket_Price)
FROM
Ski_Fields

SELECT
MIN (Ticket_Price)
FROM
Ski_Fields
```

```sql
-- Visitors
SELECT
Season_Pass_Visitors
FROM
Ski_Fields
WHERE
Season_Pass_Visitors < 0

SELECT
MAX (Season_Pass_Visitors)
FROM
Ski_Fields

SELECT
MIN (Season_Pass_Visitors)
FROM
Ski_Fields

SELECT
Multi_Day_Visitors
FROM
Ski_Fields
WHERE
Multi_Day_Visitors < 0

SELECT
Day_Ticket_Visitors
FROM
Ski_Fields
WHERE
Day_Ticket_Visitors < 0
```

</details>

## 3. Uniqueness

Completeness and validity look at individual data points, uniqueness aims to inspect whole rows compared to each other to check for repeats. This is important to do, because to answer the central question one would add each resort's visitors across all weeks, so if a week appears twice, that week's visitors are counted twice, skewing results. I used two checks: 1) Check for duplicate recordID's. I found 9 instances of repeating records. 2) Confirm that the 9 repeated ID's are exact copies. I found that they are, so these 9 rows should be removed from the data.

<details>
<summary>SQL for this check</summary>

```sql
-- Which RecordIDs appear more than once?
SELECT
RecordID,
COUNT (*) AS "Instances"
FROM
Ski_Fields
GROUP BY
RecordID
HAVING
COUNT(*) > 1
```

```sql
-- Are the repeats identical in every column?
SELECT
  RecordID, Resort, WeekStart,
  COUNT(*) AS "duplicates"
FROM
  Ski_Fields
GROUP BY
  RecordID, Resort, Location, WeekStart, Status, Snowfall,
  Ticket_Price, Season_Pass_Visitors, Multi_Day_Visitors, Day_Ticket_Visitors
HAVING
  COUNT(*) > 1
```

</details>

## 4. Consistency

Consistency aims to answer if values that share meaning are recorded the same way. I used two checks: 1) Are resort names written differently? I found that several resorts are named twice in the data, and at a glance this seemed to be because of differences in capitalization. I made sure capitalization is the only difference in naming by putting the resorts with capitalized and uncapitalized letters next to each other. 2) A resort that's closed shouldn't have visitors, and one that's open should have some. Do visitor and status counts agree? I found that they do. Consistency overall fails, as only check two passed, and check one presented 12 resorts underneath 26 spellings.

<details>
<summary>SQL for this check</summary>

```sql
-- Check 1: every spelling of every resort name
SELECT DISTINCT
Resort
FROM
Ski_Fields
```

```sql
-- Check 1: the same list, with spellings of each resort side by side
SELECT DISTINCT
Resort
FROM
Ski_Fields
ORDER BY
UPPER(Resort)
```

```sql
-- Check 2: closed weeks that report visitors
SELECT
Season_Pass_Visitors,
Multi_Day_Visitors,
Day_Ticket_Visitors,
Status
FROM
Ski_Fields
WHERE
Status = 'Closed' AND (Season_Pass_Visitors > 0 OR Multi_Day_Visitors > 0 OR Day_Ticket_Visitors > 0)
```

```sql
-- Check 2: open weeks with zero visitors
SELECT
Season_Pass_Visitors,
Multi_Day_Visitors,
Day_Ticket_Visitors,
Status
FROM
Ski_Fields
WHERE
Status = 'Open'
AND
Season_Pass_Visitors = 0
AND
Multi_Day_Visitors = 0
AND Day_Ticket_Visitors = 0
```

</details>

## 5. Reasonableness

Does visitation data actually make sense in the real world? I did a quick check to make sure that there were no unbelievable high or low visitation rates, and found that April and November were the lowest months, highest visitation is over Christmas, and none of the records seem unreasonable.

<details>
<summary>SQL for this check</summary>

```sql
-- Total visitors for each week of the season
SELECT
  WeekStart,
  SUM(Season_Pass_Visitors + Multi_Day_Visitors + Day_Ticket_Visitors) AS "Total_Visitors"
FROM
  Ski_Fields
GROUP BY
  WeekStart
ORDER BY
  Total_Visitors DESC
```

</details>

## Data cleaning

Compiling everything I found, I removed the 9 exact duplicate rows, combined the 26 variations of resort name spellings into 12 resorts, and converted Snowfall and Ticket_Price from strings to integers. Now, 12 resorts exist rather than 26, and the min/max Snowfall and Ticket_Price are accurate. In the raw data, Keystone ranked as #3 in visitation, and Steamboat ranked as #6. In the cleaned data, Steamboat now ranks as #3 and Keystone as #6. Total visitors for Breckenridge increased from 237,509 to 316,419, and total visitors decreased from 2,498,544 to 2,413,704. Non-consistent naming made each resort's total visitors too low, and duplicate entries made the overall total too high.

So, if someone were to use the uncleaned original data to understand which Colorado resorts drew the most visitors in the 2025-2026 ski season, they should not feel confident in its integrity.

```sql
-- The cleaned version of the data, saved as a view
CREATE VIEW vCleanSkiFields AS
SELECT DISTINCT
  RecordID,
  UPPER(Resort) AS Resort,
  Location,
  WeekStart,
  Status,
  CASE
    WHEN Snowfall = '' THEN NULL
    ELSE CAST(REPLACE(Snowfall, ' in', '') AS INTEGER)
  END AS Snowfall,
  CAST(REPLACE(Ticket_Price, '$', '') AS INTEGER) AS Ticket_Price,
  Season_Pass_Visitors,
  Multi_Day_Visitors,
  Day_Ticket_Visitors
FROM
  Ski_Fields;
```

<details>
<summary>SQL to check the cleaned data</summary>

```sql
-- Prove the string conversion worked
SELECT
  COUNT(*) AS Total_Rows,
  MIN(Snowfall), MAX(Snowfall),
  MIN(Ticket_Price), MAX(Ticket_Price)
FROM
  vCleanSkiFields
```

```sql
-- Before cleaning: total visitors by resort
SELECT
  Resort,
  SUM(Season_Pass_Visitors + Multi_Day_Visitors + Day_Ticket_Visitors) AS Total_Visitors
FROM
  Ski_Fields
GROUP BY
  Resort
ORDER BY
  Total_Visitors DESC
```

```sql
-- After cleaning: the same query on the cleaned view
SELECT
  Resort,
  SUM(Season_Pass_Visitors + Multi_Day_Visitors + Day_Ticket_Visitors) AS Total_Visitors
FROM
  vCleanSkiFields
GROUP BY
  Resort
ORDER BY
  Total_Visitors DESC
```

</details>
