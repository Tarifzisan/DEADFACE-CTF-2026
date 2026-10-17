# Schedule of Concern — Writeup

**Platform:** DEADFACE CTF  
**Category:** SQL / Database  
**Difficulty:** Easy  
**Points:** 20  
**Challenge Author:** @syyntax  
**Flag:** `deadface{9}`

## Challenge Overview

The challenge provides access to Aurora Health's MySQL database and asks us to determine how many drugs in the pharmacy formulary belong to **DEA Schedule II**.

Our objective is to inspect the database, locate the `drugs` table, identify the appropriate column, and count all records classified as Schedule II.

## Solution

### Step 1: Explore Available Databases

First, I listed the available databases using:

```sql
SHOW DATABASES;
```

The database server returned several databases, including:

- `aurora_audit`
- `aurora_ehr`
- `aurora_pharmacy`
- `aurora_trials`

Since the challenge specifically mentions a pharmacy formulary, `aurora_pharmacy` was the relevant database.

### Step 2: Select the Pharmacy Database

I selected the target database:

```sql
USE aurora_pharmacy;
```

Then, I listed its available tables:

```sql
SHOW TABLES;
```

The database contained five tables:

- `dispenses`
- `drugs`
- `pharmacists`
- `prescribers`
- `prescriptions`

The `drugs` table was the primary target.

### Step 3: Inspect the Drugs Table

To understand the table structure and identify the column containing DEA scheduling information, I executed:

```sql
DESCRIBE drugs;
```

The output revealed the following relevant column:

```text
dea_schedule | varchar(3)
```

Initially, I tried using `schedule` as the column name, but MySQL returned an unknown column error. Inspecting the schema helped identify the correct column: `dea_schedule`.

### Step 4: Count Schedule II Drugs

With the correct column identified, I executed the following SQL query:

```sql
SELECT COUNT(*)
FROM drugs
WHERE dea_schedule = 'II';
```

**Query Result:**

```text
+----------+
| COUNT(*) |
+----------+
|        9 |
+----------+
```

The query returned **9**, meaning there are nine drugs classified under DEA Schedule II in the pharmacy formulary.

## Final Flag

```text
deadface{9}
```

## Key Takeaways

- `SHOW DATABASES;` lists available databases.
- `USE database_name;` selects a database.
- `SHOW TABLES;` lists tables in the selected database.
- `DESCRIBE table_name;` reveals a table's schema.
- `COUNT(*)` counts records matching a specified condition.
- Always verify column names before writing SQL queries.

**Challenge Status:** Solved ✅
