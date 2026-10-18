# Upon the Throne 

**Platform:** DEADFACE CTF 2026  
**Category:** SQL / Database  
**Challenge Author:** @syyntax  
**Difficulty:** Intermediate

## Challenge Description

Aurora's breach involved two insiders: an audit-system clerk who accessed a gubernatorial candidate's medical records and an EHR physician who was involved in falsifying the Veltrexa clinical trial.

Both insiders were controlled by the same DEADFACE handler. The objective was to identify the handler's codename by finding the only `handler_ref` shared between the `users` and `providers` tables.

## Solution

### 1. Understanding the Objective

The challenge provided access to a MySQL shell and indicated that the handler's codename was stored in the `handler_ref` field of two separate tables:

- `users` — Audit system
- `providers` — Electronic Health Record (EHR) system

There were approximately fifteen different handler references in each table. Instead of checking them manually, I used an SQL `JOIN` to identify the common value.

### 2. Finding the Common Handler

I executed the following SQL query:

```sql
SELECT
    u.handler_ref,
    COUNT(DISTINCT u.handler_ref) AS audit_match
FROM aurora_audit.users AS u
JOIN aurora_ehr.providers AS p
    ON u.handler_ref = p.handler_ref
GROUP BY u.handler_ref;
```

### 3. Query Output

The database returned:

```text
+-------------+-------------+
| handler_ref | audit_match |
+-------------+-------------+
| REVENANT-7  |           1 |
+-------------+-------------+
1 row in set
```

The query returned only one matching `handler_ref` across both tables.

Therefore, the common DEADFACE handler's codename was:

**REVENANT-7**

## Flag

```text
deadface{REVENANT-7}
```

## Key Takeaway

This challenge demonstrated how SQL `INNER JOIN` can correlate information across separate databases and identify shared records.

By matching the `handler_ref` field between the audit users and EHR providers tables, I was able to identify the single operative responsible for managing insiders on both sides of Aurora's breach.
