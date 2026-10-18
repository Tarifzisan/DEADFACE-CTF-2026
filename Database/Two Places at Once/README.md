# Two Places at Once

**Platform:** DEADFACE CTF 2026  
**Category:** Database / SQL  
**Author:** @syyntax

## Challenge Description

Aurora Health's incident-response team discovered that a legitimate account was being used from two different workstations at exactly the same timestamp.

The objective was to identify the compromised account by analyzing the audit logs and finding repeated simultaneous access events.

## Solution

### 1. Inspect the Audit Database

First, I listed the available tables:

```sql
SHOW TABLES FROM aurora_audit;
```

The database contained two relevant tables:

- `access_log`
- `users`

### 2. Examine the Database Schema

I queried INFORMATION_SCHEMA to identify the relevant columns:

```sql
SELECT
    TABLE_NAME,
    COLUMN_NAME,
    DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'aurora_audit'
ORDER BY TABLE_NAME, ORDINAL_POSITION;
```

Important columns identified:

**access_log:**
- `user_id`
- `workstation`
- `accessed_at`
- `ip_address`

**users:**
- `user_id`
- `username`

### 3. Find Simultaneous Logins

I joined the `access_log` and `users` tables using `user_id`.

Then I grouped the records by user and timestamp, counting the number of distinct workstations.

```sql
SELECT
    u.username,
    a.user_id,
    a.accessed_at,
    COUNT(DISTINCT a.workstation) AS workstation_count,
    GROUP_CONCAT(DISTINCT a.workstation) AS workstations
FROM aurora_audit.access_log a
JOIN aurora_audit.users u
    ON a.user_id = u.user_id
GROUP BY
    a.user_id,
    u.username,
    a.accessed_at
HAVING COUNT(DISTINCT a.workstation) >= 2
ORDER BY workstation_count DESC, u.username;
```

### 4. Query Results

The query returned eight suspicious events, all belonging to the same account:

```text
| username | user_id | accessed_at         | workstation_count | workstations  |
|----------|---------|---------------------|-------------------|---------------|
| kbhatt   | 16      | 2025-11-04 23:54:03 | 2                 | WS-008,WS-028 |
| kbhatt   | 16      | 2025-11-05 23:55:55 | 2                 | WS-013,WS-025 |
| kbhatt   | 16      | 2025-11-14 02:18:01 | 2                 | WS-002,WS-023 |
| kbhatt   | 16      | 2025-11-18 01:23:54 | 2                 | WS-015,WS-027 |
| kbhatt   | 16      | 2025-11-19 02:37:21 | 2                 | WS-008,WS-026 |
| kbhatt   | 16      | 2025-11-21 02:51:31 | 2                 | WS-018,WS-029 |
| kbhatt   | 16      | 2025-11-23 02:29:51 | 2                 | WS-006,WS-030 |
| kbhatt   | 16      | 2025-11-27 03:57:17 | 2                 | WS-011,WS-022 |
```

### 5. Confirm the Repeated Events

To confirm that the suspicious behavior happened repeatedly, I ran:

```sql
SELECT
    u.username,
    COUNT(*) AS simultaneous_login_events
FROM (
    SELECT
        user_id,
        accessed_at
    FROM aurora_audit.access_log
    GROUP BY user_id, accessed_at
    HAVING COUNT(DISTINCT workstation) >= 2
) AS suspicious
JOIN aurora_audit.users u
    ON suspicious.user_id = u.user_id
GROUP BY u.username
HAVING COUNT(*) > 1
ORDER BY simultaneous_login_events DESC;
```

Output:

```text
+----------+---------------------------+
| username | simultaneous_login_events |
+----------+---------------------------+
| kbhatt   |                         8 |
+----------+---------------------------+
```

The account `kbhatt` had eight separate timestamps where it was used from two different workstations simultaneously.

This strongly indicated that the account had been compromised.

## Flag

```text
deadface{kbhatt}
```

## Key Takeaways

- Used INFORMATION_SCHEMA to inspect an unfamiliar database.
- Joined audit logs with user records.
- Used `COUNT(DISTINCT workstation)` to detect simultaneous access.
- Grouped events by `user_id` and `accessed_at`.
- Confirmed repeated suspicious activity using a subquery.

The compromised account was identified as `kbhatt`.
