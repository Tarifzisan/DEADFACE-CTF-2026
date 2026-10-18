# The Original Reading — 

**Challenge:** The Original Reading (70 pts)  
**Category:** Database / SQL  
**Author:** @syyntax  
**Event:** DEADFACE CTF  
**Database:** MySQL 8.0.46  
**Flag:** `deadface{2.3}`

## Challenge Description

Aurora Pharmaceuticals conducted a Phase III clinical trial for a drug called Veltrexa. The trial's Electronic Data Capture (EDC) system maintains an audit trail of every modification in a `data_changes` table.

Among the routine source-data corrections, one modification secretly altered the primary efficacy endpoint to make the drug appear more effective.

The objective was to identify the original value that was overwritten by investigating the database audit trail.

## Solution

### 1. Enumerating Available Databases

After connecting to the provided MySQL shell, I listed the available databases:

```sql
SHOW DATABASES;
```

The output revealed several databases:

- `aurora_audit`
- `aurora_ehr`
- `aurora_pharmacy`
- `aurora_trials`

Since the challenge involved clinical trial data and its modification history, I investigated `aurora_trials`.

### 2. Finding the Relevant Table

I selected the database and listed its tables:

```sql
USE aurora_trials;

SHOW TABLES;
```

The database contained the following tables:

- `adverse_events`
- `data_changes`
- `results`
- `sites`
- `studies`
- `subjects`

The `data_changes` table was particularly interesting because it contained the audit trail of database modifications.

### 3. Inspecting the Audit Table Structure

To understand the available fields, I executed:

```sql
DESCRIBE data_changes;
```

The table contained several important columns:

| Column | Description |
|---|---|
| `change_id` | Unique modification ID |
| `study_id` | Associated study |
| `table_name` | Modified table |
| `record_ref` | Reference to the modified record |
| `field_name` | Modified field |
| `old_value` | Original value before modification |
| `new_value` | Updated value |
| `changed_by` | User responsible for the change |
| `changed_at` | Modification timestamp |
| `reason` | Recorded reason for modification |

The `old_value` and `new_value` columns were particularly important for recovering the overwritten data.

### 4. Investigating the Audit Trail

I retrieved all recorded changes, sorted chronologically:

```sql
SELECT *
FROM data_changes
ORDER BY changed_at;
```

Most entries were routine source-data corrections involving subject records.

However, one entry immediately stood out:

```text
+-----------+----------+------------+------------+------------------------+-----------+-------------+------------+---------------------+------------------------------+
| change_id | study_id | table_name | record_ref | field_name             | old_value | new_value   | changed_by | changed_at          | reason                       |
+-----------+----------+------------+------------+------------------------+-----------+-------------+------------+---------------------+------------------------------+
| 500001    | 1        | results    | VELT-0137  | primary_efficacy_delta | 2.3       | 6.0         | mreyes     | 2025-12-06 23:14:00 | Query resolution per monitor |
+-----------+----------+------------+------------+------------------------+-----------+-------------+------------+---------------------+------------------------------+
```

Unlike the other modifications, this entry changed a result-related field named `primary_efficacy_delta`.

The value had been changed from `2.3` to `6.0`.

This matched the challenge's description of an altered primary efficacy endpoint.

### 5. Identifying the Original Value

The relevant record contained:

- **Record Reference:** `VELT-0137`
- **Modified Field:** `primary_efficacy_delta`
- **Original Value:** `2.3`
- **New Value:** `6.0`
- **Changed By:** `mreyes`
- **Modification Time:** `2025-12-06 23:14:00`

The original value was preserved in the `old_value` column of the audit trail.

Therefore, the overwritten value was `2.3`.

## Flag

```text
deadface{2.3}
```

## Key Takeaway

This challenge demonstrates the importance of database audit trails in detecting unauthorized or suspicious modifications.

Even when a database record has been updated, historical values can remain recoverable if the system properly maintains an audit log.

**Lesson:** Never overlook `old_value` when investigating database modifications. It may contain exactly the evidence needed to reconstruct what was changed.
