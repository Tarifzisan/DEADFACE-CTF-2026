# Down the Rabbit Hole

**Challenge:** Down the Rabbit Hole  
**Category:** Mobile App / Steganography  
**Points:** 10  
**CTF:** DEADFACE CTF 2026

**Flag:** `deadface{847261}`

## 1. Challenge Objective

This challenge reuses the application files from the previous **Info Leaks** challenge.

The Aurora Patient Portal application uses a unique Patient ID to associate patient information with medical records. Our objective is to identify which file inside the application bundle hides this identifier.

The expected flag format is:

```text
deadface{ID}
```

As with the previous challenge, no emulator or debugger is required. Static analysis is sufficient.

## 2. Identifying the Likely Carrier

From the extracted IPA bundle, one file immediately stands out:

```text
aurora_chest_xray.jpg
```

This JPEG image appears to be a demo imaging asset bundled with the application.

Further inspection of the Mach-O executable reveals references to this image alongside a string:

```text
Demo chart for Patient:
```

This suggests that the image may be associated with a specific patient record.

Since the challenge focuses on hiding a Patient ID, inspecting the image's metadata is a reasonable next step.

## 3. Analyzing JPEG Metadata

JPEG files can contain several metadata segments, including:

- EXIF — camera and image-related metadata.
- IPTC — descriptive and administrative information.
- XMP — extensible metadata that can store structured information.

Inspecting the image's metadata and examining its embedded strings reveals that the JPEG contains EXIF, Photoshop Image Resource Block (IRB), and XMP segments.

The XMP packet contains an interesting Dublin Core identifier:

```xml
<dc:identifier>PID-847261</dc:identifier>
```

This is particularly interesting because the identifier follows a Patient ID-style naming convention.

The extracted identifier is:

```text
PID-847261
```

No additional data is appended after the JPEG End of Image (EOI) marker. Therefore, the XMP metadata provides the relevant hidden value.

## 4. Cross-Referencing with the SQLite Database

The application bundle also contains another potentially useful file:

```text
aurora_meds.sqlite
```

Opening the SQLite database reveals a table named:

```text
medications
```

The table contains medication records associated with patient identifiers. Its primary key is `pid`.

To verify whether the extracted identifier corresponds to an actual database record, execute:

```sql
SELECT *
FROM medications
WHERE pid = 847261;
```

The query returns:

```text
847261 | Ibuprofen | 400 mg | Every 8 hours as needed | Candesartan | 16 mg | Once daily
```

The database contains 100 records, and `847261` is the highest ID.

Most importantly, the Patient ID extracted from the JPEG's XMP metadata matches an existing record in the medication database.

This cross-reference confirms that `847261` is the Patient ID associated with the bundled medical record.

## 5. Flag

The extracted XMP identifier is:

```text
PID-847261
```

The database confirms that its numeric portion corresponds to a valid patient record.

Following the requested flag format, the answer is:

```text
deadface{847261}
```

**Alternative:** If the platform expects the complete metadata identifier rather than its numeric portion, try:

```text
deadface{PID-847261}
```

## 6. Root Cause and Security Impact

The application exposes a patient identifier through the XMP metadata of an image bundled inside the IPA.

Although the image appears to be an ordinary chest X-ray asset, its metadata contains a unique identifier that can be extracted without launching the application.

The risk becomes more significant because the same Patient ID is used as a key in the bundled SQLite medication database.

An attacker who obtains the IPA can:

- Extract patient identifiers from image metadata.
- Cross-reference those identifiers with bundled medical records.
- Associate patient IDs with medication information.
- Combine the findings with personal information exposed in the previous Info Leaks challenge.

Together, these weaknesses allow an attacker to build a more complete patient profile, increasing the effectiveness of targeted phishing campaigns.

## 7. Remediation

Several security improvements could prevent this type of exposure:

1. **Strip image metadata:** Remove EXIF, XMP, and IPTC metadata before including images in production application bundles.

2. **Avoid embedding patient identifiers:** Never store real patient IDs inside image metadata, filenames, or other publicly accessible assets.

3. **Remove patient-keyed databases:** Do not ship production patient records or sensitive demo databases inside the application package.

4. **Use server-side record lookup:** Retrieve medical records through authenticated and authorized API requests instead of exposing patient-keyed databases to clients.

5. **Apply data minimization:** Use non-sensitive, short-lived identifiers on the client wherever possible.

6. **Automate security scanning:** Add CI/CD checks to inspect image metadata, SQLite databases, and other bundled resources for sensitive identifiers, including patterns such as `PID-\d+`.

## Conclusion

This challenge demonstrates that sensitive information can hide in places developers often overlook.

By inspecting the bundled JPEG's XMP metadata, extracting the embedded Patient ID, and cross-referencing it against the SQLite medication database, we successfully identified the patient identifier through static analysis alone.

**Key takeaway:** Image metadata is part of the attack surface. A harmless-looking image can expose identifiers that unlock access to much more sensitive information.
