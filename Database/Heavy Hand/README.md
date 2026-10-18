# Heavy Hand 

**Challenge:** Heavy Hand (100 points)  
**Category:** Database / SQL  
**Author:** @syyntax  
**Event:** DEADFACE CTF

## Challenge Description

One Aurora prescriber dispensed significantly more Schedule II medication than anyone else during the final quarter of 2025. The objective was to identify the prescriber with the highest total quantity of Schedule II medication dispensed between October 1 and December 31, 2025, and retrieve their DEA number.

**Flag Format:** `deadface{DEA}`

## Solution

### 1. Database Enumeration

First, I connected to the provided MySQL shell and selected the target database.

```sql
USE aurora_pharmacy;
```

Listed the available tables:

```sql
SHOW TABLES;
```

The database contained five tables:

- `dispenses`
- `drugs`
- `pharmacists`
- `prescribers`
- `prescriptions`

### 2. Table Structure Analysis

I inspected the relevant tables to understand their relationships.

```sql
DESCRIBE dispenses;
DESCRIBE drugs;
DESCRIBE prescribers;
DESCRIBE prescriptions;
```

The important columns were:

| Table | Relevant Columns |
|---|---|
| `dispenses` | `rx_id`, `quantity_dispensed`, `dispensed_at` |
| `drugs` | `drug_id`, `dea_schedule` |
| `prescriptions` | `rx_id`, `prescriber_id`, `drug_id` |
| `prescribers` | `prescriber_id`, `dea_number`, `first_name`, `last_name` |

The relationships were:

- `dispenses.rx_id` → `prescriptions.rx_id`
- `prescriptions.drug_id` → `drugs.drug_id`
- `prescriptions.prescriber_id` → `prescribers.prescriber_id`

### 3. Identifying the Highest-Volume Prescriber

I joined the four tables to connect dispensing records with their corresponding prescriptions, medications, and prescribers.

The query filters for Schedule II medications and dispensing dates within Q4 2025. It then calculates the total quantity dispensed by each prescriber.

```sql
SELECT
    p.prescriber_id,
    p.first_name,
    p.last_name,
    p.dea_number,
    SUM(d.quantity_dispensed) AS total_quantity
FROM dispenses d
JOIN prescriptions rx
    ON d.rx_id = rx.rx_id
JOIN drugs dr
    ON rx.drug_id = dr.drug_id
JOIN prescribers p
    ON rx.prescriber_id = p.prescriber_id
WHERE dr.dea_schedule = 'II'
  AND d.dispensed_at >= '2025-10-01'
  AND d.dispensed_at < '2026-01-01'
GROUP BY
    p.prescriber_id,
    p.first_name,
    p.last_name,
    p.dea_number
ORDER BY total_quantity DESC
LIMIT 1;
```

### 4. Query Result

The query returned:

| Field | Result |
|---|---|
| Prescriber ID | 2 |
| First Name | Marcus |
| Last Name | Reyes |
| DEA Number | BR2938471 |
| Total Quantity Dispensed | 5,220 |

Marcus Reyes had the highest total quantity of Schedule II medication dispensed during the specified period, with a total of **5,220 units**.

## Flag

```text
deadface{BR2938471}
```

## Key Takeaways

- Used SQL `JOIN` operations to correlate multiple database tables.
- Used `SUM()` to calculate aggregate dispensing quantities.
- Applied date filtering using an inclusive start date and exclusive end date.
- Used `GROUP BY` and `ORDER BY DESC` to identify the highest-volume prescriber.

**Challenge Solved!**
