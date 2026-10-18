# SQL Skillz 

**Challenge:** SQL Skillz  
**Category:** Mobile App / SQL  
**Points:** 50  
**CTF:** DEADFACE CTF 2026

**Flag:** `deadface{Candesartan}`

## 1. Challenge Objective

This challenge continues the investigation of the Aurora Patient Portal iOS application from the previous two challenges:

- **Info Leaks:** Revealed the patient's name and date of birth.
- **Down the Rabbit Hole:** Extracted the Patient ID from the XMP metadata of a bundled JPEG image.

Now, the objective is to query the application's SQLite database and identify which medication the current patient takes besides Ibuprofen.

The expected flag format is:

```text
deadface{Name}
```

## 2. Identifying the Database

During the initial IPA analysis, we discovered a SQLite database inside the application bundle:

```text
aurora_meds.sqlite
```

The database contains a table named:

```text
medications
```

The table uses `pid` as its primary key, allowing medication records to be associated with individual patients.

From the previous challenge, we already identified the Patient ID:

```text
847261
```

This identifier provides the key needed to retrieve the patient's medication record.

## 3. Querying the Database

Using SQLite, execute the following query:

```sql
SELECT *
FROM medications
WHERE pid = 847261;
```

The query returns:

```text
847261 | Ibuprofen | 400 mg | Every 8 hours as needed | Candesartan | 16 mg | Once daily
```

The record contains two medications:

| Medication | Dosage | Frequency |
|---|---|---|
| Ibuprofen | 400 mg | Every 8 hours as needed |
| Candesartan | 16 mg | Once daily |

The challenge specifically asks for the medication other than Ibuprofen.

Therefore, the answer is:

```text
Candesartan
```

## 4. Flag

Following the required flag format:

```text
deadface{Candesartan}
```

## 5. Root Cause and Security Impact

The underlying issue is the inclusion of a readable, unencrypted SQLite database containing patient-specific medication information inside the distributed iOS application.

Because the database is keyed by the same Patient ID exposed through the JPEG's XMP metadata, an attacker can combine information from multiple application assets to retrieve sensitive medical details.

Potential consequences include:

- **Medical Information Exposure:** Unauthorized individuals can access patient medication records.
- **Privacy Violation:** Patient identifiers can be correlated with sensitive medical information.
- **Targeted Phishing:** Attackers can use medication details to make fraudulent communications appear more legitimate.

This challenge demonstrates how individually exposed data points can be combined to reveal more sensitive information.

## 6. Remediation

1. **Remove patient databases from application bundles:** Never distribute patient-specific medical records inside publicly accessible IPA files.

2. **Use authenticated backend APIs:** Retrieve medication information from a secure server after authentication and authorization checks.

3. **Encrypt offline data:** If offline access is necessary, use appropriately encrypted local storage with iOS Data Protection.

4. **Scope cached data to the signed-in patient:** Ensure locally stored records cannot be accessed across different user sessions.

5. **Clear sensitive data on sign-out:** Remove cached patient records when the user signs out or their session expires.

6. **Apply least privilege:** Ensure users can only retrieve medical records they are authorized to access.

## Conclusion

This challenge demonstrates the risks of distributing sensitive patient databases inside mobile application packages.

By reusing the Patient ID discovered in the previous challenge and querying the bundled SQLite database, we identified Candesartan as the patient's medication other than Ibuprofen.

**Key takeaway:** A patient identifier combined with an exposed database can turn a seemingly minor information leak into a significant medical privacy breach.
