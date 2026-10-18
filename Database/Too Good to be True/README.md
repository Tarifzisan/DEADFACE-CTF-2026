# Too Good to be True 
**Platform:** DEADFACE CTF 2026  
**Category:** Database / SQL  
**Author:** @syyntax

## Challenge Description

The challenge involved investigating fabricated clinical trial results in Aurora's Veltrexa study.

One subject had an efficacy improvement that was statistically impossible, exceeding three standard deviations above the study-wide mean.

The objective was to identify the outlier subject and retrieve their subject code.

## Solution

### 1. Identify the Trial Database

First, I listed the available databases:

```sql
SHOW DATABASES;
```

I found the `aurora_trials` database.

Next, I listed its tables:

```sql
SHOW TABLES FROM aurora_trials;
```

The relevant tables were:

- `studies`
- `subjects`
- `results`

### 2. Inspect the Database Schema

I queried INFORMATION_SCHEMA to identify the relevant columns:

```sql
SELECT
    TABLE_NAME,
    COLUMN_NAME,
    DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'aurora_trials'
ORDER BY TABLE_NAME, ORDINAL_POSITION;
```

The `results` table contained the following important fields:

- `subject_id`
- `measure`
- `visit`
- `value`

The `subjects` table contained `subject_code` and `study_id`.

### 3. Identify the Veltrexa Study

I searched for the Veltrexa clinical trial:

```sql
SELECT
    study_id,
    protocol_number,
    title,
    drug_name
FROM aurora_trials.studies
WHERE drug_name LIKE '%Veltrexa%'
   OR title LIKE '%Veltrexa%';
```

The result identified:

- Study ID: `1`
- Protocol: `AUR-VLT-3001`
- Drug: Veltrexa (velcoraxine)

### 4. Identify the Primary Endpoint

Next, I checked the available measurements:

```sql
SELECT
    measure,
    visit,
    unit,
    COUNT(*) AS total_records
FROM aurora_trials.results
GROUP BY measure, visit, unit
ORDER BY measure, visit;
```

The relevant measurement was:

```text
primary_efficacy_delta | Week 24 | points | 60
```

This was the primary efficacy endpoint used for the investigation.

### 5. Calculate Statistical Outliers

To identify subjects exceeding three standard deviations above the mean, I used `AVG()` and `STDDEV_POP()`.

The SQL query:

```sql
SELECT
    s.subject_code,
    r.value AS efficacy_improvement,
    ROUND(stats.mean_value, 4) AS study_mean,
    ROUND(stats.sd_value, 4) AS standard_deviation,
    ROUND(
        (r.value - stats.mean_value) / stats.sd_value,
        4
    ) AS z_score
FROM aurora_trials.results r
JOIN aurora_trials.subjects s
    ON r.subject_id = s.subject_id
CROSS JOIN (
    SELECT
        AVG(r2.value) AS mean_value,
        STDDEV_POP(r2.value) AS sd_value
    FROM aurora_trials.results r2
    JOIN aurora_trials.subjects s2
        ON r2.subject_id = s2.subject_id
    WHERE s2.study_id = 1
      AND r2.measure = 'primary_efficacy_delta'
      AND r2.visit = 'Week 24'
) AS stats
WHERE s.study_id = 1
  AND r.measure = 'primary_efficacy_delta'
  AND r.visit = 'Week 24'
  AND r.value > stats.mean_value + 3 * stats.sd_value;
```

### 6. Query Output

The database returned:

```text
+--------------+----------------------+------------+--------------------+---------+
| subject_code | efficacy_improvement | study_mean | standard_deviation | z_score |
+--------------+----------------------+------------+--------------------+---------+
| VELT-0148    |                22.00 |     2.9650 |             2.8067 |   6.782 |
+--------------+----------------------+------------+--------------------+---------+
```

The subject `VELT-0148` had an efficacy improvement of 22.00 points, with a Z-score of 6.782.

Since this value exceeded the three-standard-deviation threshold, it was identified as the statistical outlier.

## Flag

```text
deadface{VELT-0148}
```

## Key Takeaways

- Used INFORMATION_SCHEMA to investigate an unfamiliar database schema.
- Identified the relevant clinical trial and primary endpoint.
- Used SQL aggregate functions to calculate the mean and population standard deviation.
- Applied Z-score analysis to identify an extreme statistical outlier.

The investigation revealed that subject `VELT-0148` had an unusually high recorded efficacy improvement.
