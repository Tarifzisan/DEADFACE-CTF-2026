# Ghost in the Registry 

**Category:** Database / SQL  
**Points:** 240  
**Author:** @syyntax  
**Challenge:** Ghost in the Registry

**Flag:** `deadface{AH-0090001}`

## Challenge Description

The challenge describes a suspected fabricated patient who receives controlled-substance prescriptions but has never attended a clinical encounter.

Aurora uses a shared `patient_id` across its enterprise systems, allowing patient records to be correlated between databases.

The objective is to identify a patient who:
- Has at least one Schedule II prescription.
- Has zero clinical encounters in `aurora_ehr`.

The final answer must be the patient's MRN.

## 1. Database Investigation

I accessed the provided MySQL shell and investigated the relevant tables across the Aurora databases.

The important tables were:

- `aurora_ehr.patients` — Contains patient information, including `patient_id` and MRN.
- `aurora_ehr.encounters` — Contains clinical encounter records.
- `aurora_pharmacy.prescriptions` — Contains prescription records.
- `aurora_pharmacy.drugs` — Contains drug information, including DEA schedule classifications.

Since both systems share the same `patient_id`, the records can be correlated using SQL joins.

## 2. Constructing the SQL Query

I joined the patient, prescription, and drug tables to identify patients with Schedule II prescriptions.

To exclude patients with clinical encounters, I used a `NOT EXISTS` subquery against `aurora_ehr.encounters`.

The final query was:

```sql
SELECT
    p.patient_id,
    p.mrn,
    COUNT(DISTINCT rx.rx_id) AS schedule_ii_prescriptions
FROM aurora_ehr.patients p
JOIN aurora_pharmacy.prescriptions rx
    ON p.patient_id = rx.patient_id
JOIN aurora_pharmacy.drugs dr
    ON rx.drug_id = dr.drug_id
WHERE dr.dea_schedule = 'II'
AND NOT EXISTS (
    SELECT 1
    FROM aurora_ehr.encounters e
    WHERE e.patient_id = p.patient_id
)
GROUP BY p.patient_id, p.mrn;
```

### Query Explanation

1. **Patient and prescription join:** Matches patients with their pharmacy prescriptions using `patient_id`.
2. **Drug join:** Connects prescriptions to drug information.
3. **Schedule II filter:** `dr.dea_schedule = 'II'` selects Schedule II controlled substances.
4. **Encounter exclusion:** `NOT EXISTS` ensures the patient has no clinical encounters.
5. **Aggregation:** `COUNT(DISTINCT rx.rx_id)` counts the qualifying prescriptions for each patient.

## 3. Query Results

The query returned exactly one matching patient:

```text
+------------+------------+---------------------------+
| patient_id | mrn        | schedule_ii_prescriptions |
+------------+------------+---------------------------+
|       1060 | AH-0090001 |                         3 |
+------------+------------+---------------------------+
1 row in set (0.002 sec)
```

The result identifies patient `1060`, who has three Schedule II prescriptions but no recorded clinical encounters.

## 4. Flag

The patient's MRN is `AH-0090001`.

**Final Flag:**

```text
deadface{AH-0090001}
```

## Conclusion

By correlating patient records across Aurora EHR and Pharmacy databases, I identified a patient with three Schedule II prescriptions and no clinical encounter history.

The shared `patient_id` made it possible to connect records across the two systems and isolate the suspicious patient using a SQL `NOT EXISTS` condition.
