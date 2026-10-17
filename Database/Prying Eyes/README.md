# Prying Eyes — DEADFACE CTF 2026 (Database / SQL, 55 Points)

**Challenge:** Prying Eyes  
**Category:** Database / SQL  
**Difficulty:** Easy  
**Author:** @syyntax  
**Flag:** `deadface{rborden}`

## Challenge Description

Aurora Health logs every time a staff member opens a patient chart. Someone was monitoring a VIP patient unusually closely.

The objective was to identify the user account that accessed the VIP patient's chart more times than any other account.

The VIP patient's MRN was obtained from the previous challenge, **Admitted Under Another Name**.

## 1. Database Enumeration

After accessing the provided MySQL shell, I initially tried listing tables but encountered an error because no database was selected.

```sql
SHOW DATABASES;
```

The database server contained several databases:

- `aurora_audit`
- `aurora_ehr`
- `aurora_pharmacy`
- `aurora_trials`

Since the challenge involved patient chart access logs, I selected the audit database.

```sql
USE aurora_audit;
SHOW TABLES;
```

The database contained two tables:

```text
access_log
users
```

## 2. Understanding the Database Schema

I inspected both tables to understand how access records were stored and how they could be associated with user accounts.

```sql
DESCRIBE access_log;
```

Important columns in `access_log`:

| Column | Description |
|---|---|
| `log_id` | Unique log identifier |
| `user_id` | ID of the accessing user |
| `patient_mrn` | Patient's Medical Record Number |
| `patient_id` | Patient identifier |
| `action` | Recorded action |
| `resource` | Accessed resource |
| `accessed_at` | Access timestamp |

Next, I inspected the `users` table:

```sql
DESCRIBE users;
```

Important columns:

| Column | Description |
|---|---|
| `user_id` | Unique user identifier |
| `username` | Account username |
| `full_name` | User's full name |
| `role` | User's role |
| `department` | Department |

The `user_id` column was the common field between the two tables, making a SQL JOIN the appropriate approach.

## 3. Identifying the VIP Patient's MRN

The challenge explicitly mentioned that the VIP patient's MRN could be obtained from the previous challenge, **Admitted Under Another Name**.

From that investigation, I identified the VIP patient as:

- **Patient:** George Harmon
- **Patient ID:** `1007`
- **VIP MRN:** `AH-0044719`

This MRN was required because the audit log referenced patients using their MRN.

## 4. Finding the Most Frequent Account

With the MRN identified, I wrote a SQL query to:

1. Join `access_log` with `users` using `user_id`.
2. Filter records belonging to the VIP patient's MRN.
3. Group the records by user account.
4. Count how many times each account accessed the chart.
5. Sort the results in descending order of access count.

```sql
SELECT
    u.username,
    u.full_name,
    COUNT(*) AS access_count
FROM aurora_audit.access_log a
JOIN aurora_audit.users u
    ON a.user_id = u.user_id
WHERE a.patient_mrn = 'AH-0044719'
GROUP BY u.user_id, u.username, u.full_name
ORDER BY access_count DESC;
```

## 5. Query Results

The query returned the following results:

| Username | Full Name | Access Count |
|---|---|---:|
| `rborden` | Rebecca Borden | 47 |
| `mreyes_ehr` | Marcus Reyes | 5 |
| `ebaker7` | Elizabeth Baker | 2 |
| `manderson12` | Mark Anderson | 2 |
| `cyoung18` | Cynthia Young | 1 |
| `csanchez6` | Cynthia Sanchez | 1 |

The results clearly showed that **Rebecca Borden (`rborden`)** accessed the VIP patient's chart 47 times, significantly more than any other account.

## 6. Flag

Therefore, the identified username was `rborden`.

```text
deadface{rborden}
```

## Conclusion

This challenge demonstrated how SQL aggregation and table relationships can be used to investigate suspicious activity in healthcare audit logs.

By combining patient MRN filtering, table joins, grouping, and record counting, I identified the account responsible for the highest number of accesses to the VIP patient's medical chart.

**Key SQL Concepts:**
- `SHOW DATABASES`
- `USE`
- `DESCRIBE`
- `JOIN`
- `WHERE`
- `GROUP BY`
- `COUNT(*)`
- `ORDER BY DESC`

**Challenge Solved!**
