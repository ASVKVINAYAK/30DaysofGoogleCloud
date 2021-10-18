# GSP3787 : Insights From Data With Bigquery Challenge Lab

**From Navigation Menu -> BigQuery->In Editor, run**

## Task 1 : Total Confirmed Cases
```bash
SELECT sum(cumulative_confirmed) as total_cases_worldwide
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date='2020-04-15'

```

## Task 2 : Worst Affected Areas
```bash
with deaths_by_states as (

    SELECT subregion1_name as state, sum(cumulative_deceased) as death_count

    FROM `bigquery-public-data.covid19_open_data.covid19_open_data`

    where country_name="United States of America" and date='2020-04-10' and subregion1_name is NOT NULL

    group by subregion1_name
)

select count(*) as count_of_states

from deaths_by_states

where death_count > 100

```

## Task 3 : Identifying Hotspots
```bash
SELECT * FROM (

    SELECT subregion1_name as state, sum(cumulative_confirmed) as total_confirmed_cases

    FROM `bigquery-public-data.covid19_open_data.covid19_open_data`

    WHERE country_code="US" AND date='2020-04-10' AND subregion1_name is NOT NULL

    GROUP BY subregion1_name

    ORDER BY total_confirmed_cases DESC
)
WHERE total_confirmed_cases > 1000

```

## Task 4 : Fatality Ratio
```bash
SELECT sum(cumulative_confirmed) as total_confirmed_cases, sum(cumulative_deceased) as total_deaths, (sum(cumulative_deceased)/sum(cumulative_confirmed))*100 as case_fatality_ratio

FROM `bigquery-public-data.covid19_open_data.covid19_open_data`

where country_name="Italy" AND date BETWEEN '2020-04-01'and '2020-04-30'

```

## Task 5 : Identifying specific day
```bash
SELECT date

FROM `bigquery-public-data.covid19_open_data.covid19_open_data`

where country_name="Italy" and cumulative_deceased>10000

order by date asc

limit 1

```

## Task 6 : Finding days with zero net new cases
```bash
WITH india_cases_by_date AS (

  SELECT

    date,

    SUM( cumulative_confirmed ) AS cases

  FROM

    `bigquery-public-data.covid19_open_data.covid19_open_data`

  WHERE

    country_name ="India"

    AND date between '2020-02-21' and '2020-03-15'

  GROUP BY

    date

  ORDER BY

    date ASC

 )

, india_previous_day_comparison AS

(SELECT

  date,

  cases,

  LAG(cases) OVER(ORDER BY date) AS previous_day,

  cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases

FROM india_cases_by_date

)

select count(*)

from india_previous_day_comparison

where net_new_cases=0

```

## Task 7 : Doubling rate
```bash
WITH us_cases_by_date AS (

  SELECT

    date,

    SUM(cumulative_confirmed) AS cases

  FROM

    `bigquery-public-data.covid19_open_data.covid19_open_data`

  WHERE

    country_name="United States of America"

    AND date between '2020-03-22' and '2020-04-20'

  GROUP BY

    date

  ORDER BY

    date ASC

 )



, us_previous_day_comparison AS

(SELECT

  date,

  cases,

  LAG(cases) OVER(ORDER BY date) AS previous_day,

  cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases,

  (cases - LAG(cases) OVER(ORDER BY date))*100/LAG(cases) OVER(ORDER BY date) AS percentage_increase

FROM us_cases_by_date

)



select Date, cases as Confirmed_Cases_On_Day, previous_day as Confirmed_Cases_Previous_Day, percentage_increase as Percentage_Increase_In_Cases

from us_previous_day_comparison

where percentage_increase > 10

```

## Task 8 : Recovery rate
```bash
WITH cases_by_country AS (

  SELECT

    country_name AS country,

    sum(cumulative_confirmed) AS cases,

    sum(cumulative_recovered) AS recovered_cases

  FROM

    bigquery-public-data.covid19_open_data.covid19_open_data

  WHERE

    date = '2020-05-10'

  GROUP BY

    country_name

 )



, recovered_rate AS

(SELECT

  country, cases, recovered_cases,

  (recovered_cases * 100)/cases AS recovery_rate

FROM cases_by_country

)



SELECT country, cases AS confirmed_cases, recovered_cases, recovery_rate

FROM recovered_rate

WHERE cases > 50000

ORDER BY recovery_rate desc

LIMIT 10

```

## Task 9 : CDGR - Cumulative Daily Growth Rate
```bash
WITH

  france_cases AS (

  SELECT

    date,

    SUM(cumulative_confirmed) AS total_cases

  FROM

    `bigquery-public-data.covid19_open_data.covid19_open_data`

  WHERE

    country_name="France"

    AND date IN ('2020-01-24',

      '2020-05-10')

  GROUP BY

    date

  ORDER BY

    date)

, summary as (

SELECT

  total_cases AS first_day_cases,

  LEAD(total_cases) OVER(ORDER BY date) AS last_day_cases,

  DATE_DIFF(LEAD(date) OVER(ORDER BY date),date, day) AS days_diff

FROM

  france_cases

LIMIT 1

)

select first_day_cases, last_day_cases, days_diff, POW((last_day_cases/first_day_cases),(1/days_diff))-1 as cdgr

from summary

```

**This Lab is Done**😊
