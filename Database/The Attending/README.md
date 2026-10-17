# The Attending (Database / SQL, 40 Points)

**Challenge:** The Attending  
**Category:** Database / SQL  
**Difficulty:** Easy  
**Author:** @syyntax  
**Flag:** `deadface{mreyes}`

## Challenge Description

Aurora's VIP patient had exactly one inpatient admission in November 2025. The objective was to identify the attending physician who signed off on that admission and retrieve their system username.

We were provided with a MySQL shell to investigate the hospital database.

## 1. Database Enumeration

First, I listed the available databases:

```sql
SHOW DATABASES;
```

Among the available databases, `aurora_ehr` appeared to contain the Electronic Health Records (EHR).

```sql
USE aurora_ehr;
SHOW TABLES;
```

The database contained the following tables:

- `clinical_notes`
- `diagnoses`
- `encounters`
- `insurance_plans`
- `patients`
- `providers`

The `patients`, `encounters`, and `providers` tables were particularly relevant to this challenge.

## 2. Inspecting the Table Structures

I examined the relevant table schemas to understand their relationships.

```sql
DESCRIBE patients;
DESCRIBE encounters;
DESCRIBE providers;
```

Important columns identified:

| Table | Relevant Columns |
|---|---|
| patients | `patient_id`, `is_vip` |
| encounters | `patient_id`, `attending_provider_id`, `encounter_type`, `admit_date` |
| providers | `provider_id`, `first_name`, `last_name`, `username` |

The `attending_provider_id` in the `encounters` table could be joined with `provider_id` in the `providers` table to identify the responsible physician.

## 3. Identifying the VIP Patient

The challenge explicitly mentioned that the target patient had `is_vip = 1`.

I queried the patients table:

```sql
SELECT *
FROM patients
WHERE is_vip = 1;
```

The query returned exactly one patient:

```text
patient_id: 1007
first_name: Gregory
last_name: Halloran
alias: George Harmon
is_vip: 1
```

The target patient's ID was therefore `1007`.

## 4. Finding the November 2025 Inpatient Admission

Next, I searched the `encounters` table for inpatient admissions belonging to patient `1007` during November 2025.

```sql
SELECT *
FROM encounters
WHERE patient_id = 1007
  AND encounter_type = 'inpatient'
  AND admit_date >= '2025-11-01'
  AND admit_date < '2025-12-01';
```

The query returned exactly one matching admission:

```text
encounter_id:          5017
patient_id:            1007
attending_provider_id: 5
encounter_type:        Inpatient
department:            Pain Clinic
admit_date:            2025-11-14
discharge_date:         2025-11-17
```

The attending physician's provider ID was `5`.

## 5. Identifying the Attending Physician

Finally, I joined the `encounters` table with the `providers` table using `attending_provider_id` and `provider_id`.

```sql
SELECT
    e.encounter_id,
    e.admit_date,
    e.discharge_date,
    p.first_name,
    p.last_name,
    p.username
FROM encounters e
JOIN providers p
    ON e.attending_provider_id = p.provider_id
WHERE e.patient_id = 1007
  AND e.encounter_type = 'inpatient'
  AND e.admit_date >= '2025-11-01'
  AND e.admit_date < '2025-12-01';
```

The result was:

```text
+--------------+------------+----------------+------------+-----------+----------+
| encounter_id | admit_date | discharge_date | first_name | last_name | username |
+--------------+------------+----------------+------------+-----------+----------+
| 5017         | 2025-11-14 | 2025-11-17     | Marcus     | Reyes     | mreyes   |
+--------------+------------+----------------+------------+-----------+----------+
```

The attending physician was **Marcus Reyes**, and his system username was `mreyes`.

## 6. Flag

```text
deadface{mreyes}
```

**Flag:** `deadface{mreyes}`

## Key Takeaways

- Enumerating databases and tables in MySQL.
- Inspecting table schemas using `DESCRIBE`.
- Filtering records using multiple `WHERE` conditions.
- Working with date ranges in SQL.
- Performing `INNER JOIN` operations to correlate records across relational tables.
- Tracing a patient's admission to the responsible physician.
