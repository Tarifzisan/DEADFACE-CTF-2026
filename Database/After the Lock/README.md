# After the Lock 

**Challenge:** After the Lock  
**Category:** Database / SQL  
**Points:** 150  
**Author:** @syyntax

**Flag:** `deadface{VELT-0137}`

## 1. Challenge Overview

Aurora Health maintains clinical trial records in a MySQL database. According to the challenge description, the Veltrexa study has an official data-lock date recorded in the `studies` table.

The objective was to identify the subject whose `primary_efficacy_delta` result was modified after the official data-lock date.

## 2. Database Enumeration

First, I checked the available databases:

```sql
SHOW DATABASES;
```

The output revealed several databases, including `aurora_trials`.

I initially attempted to access `aurora_health`, but access was denied. Since the challenge involved clinical trial records, I selected `aurora_trials` instead.

```sql
USE aurora_trials;
```

Then I listed the available tables:

```sql
SHOW TABLES;
```

The database contained six tables:

- `adverse_events`
- `data_changes`
- `results`
- `sites`
- `studies`
- `subjects`

## 3. Schema Enumeration

To understand the database structure, I queried `INFORMATION_SCHEMA.COLUMNS`:

```sql
SELECT
    TABLE_NAME,
    COLUMN_NAME,
    DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'aurora_trials'
ORDER BY TABLE_NAME, ORDINAL_POSITION;
```

Two tables were particularly important:

- `studies`: Contains study information and the official `data_lock_date`.
- `data_changes`: Contains audit records, including modified fields, old and new values, timestamps, and subject references.

The `results` and `subjects` tables were also relevant for cross-checking the affected measurement.

## 4. Finding the Veltrexa Study

I searched the `studies` table for Veltrexa:

```sql
SELECT
    study_id,
    protocol_number,
    title,
    drug_name,
    data_lock_date
FROM studies
WHERE drug_name LIKE '%Veltrexa%'
   OR title LIKE '%Veltrexa%';
```

The query returned:

| Field | Value |
|---|---|
| Study ID | 1 |
| Protocol | AUR-VLT-3001 |
| Drug | Veltrexa (velcoraxine) |
| Data-lock date | 2025-12-01 |

The official data-lock date was **December 1, 2025**.

## 5. Investigating the Audit Log

Next, I examined the audit trail for modifications involving the results table and the target measurement:

```sql
SELECT *
FROM data_changes
WHERE field_name LIKE '%primary_efficacy%'
   OR table_name = 'results'
ORDER BY changed_at;
```

This returned a suspicious record:

| Field | Value |
|---|---|
| Change ID | 500001 |
| Study ID | 1 |
| Table | results |
| Record Reference | VELT-0137 |
| Field | primary_efficacy_delta |
| Old Value | 2.3 |
| New Value | 6.0 |
| Changed By | mreyes |
| Changed At | 2025-12-06 23:14:00 |
| Reason | Query resolution per monitor |

The modification occurred on December 6, 2025, which was five days after the official data-lock date.

Therefore, this record was modified after the study was locked.

## 6. Flag

The affected subject code was `VELT-0137`.

**Flag:**

```text
deadface{VELT-0137}
```

## 7. Conclusion

By enumerating the MySQL database, identifying the Veltrexa study, and examining its audit trail, I found a modification to the `primary_efficacy_delta` measurement after the official data-lock date.

The affected subject was **VELT-0137**.
