# Info LeakS

**Challenge:** Info Leaks  
**Category:** Forensics / Mobile  
**Points:** 25  
**CTF:** DEADFACE CTF 2026  
**Author:** @G2Gh0st

**Flag:** `deadface{mara-voss-1988-04-17}`

## 1. Challenge Objective

Aurora Health patients are being targeted by phishing campaigns containing accurate personal information. Interestingly, every victim recently installed the new Aurora Patient Portal iOS application.

The objective is to identify the **name and date of birth of the patient who last used this particular app build**.

The expected flag format is:

```text
deadface{firstname-lastname-YYYY-MM-DD}
```

The challenge hint explicitly states that no emulator or debugger is required, suggesting that static analysis of the provided IPA should be sufficient.

## 2. Unpacking the IPA

An IPA (iOS App Store Package) is essentially a ZIP archive. Therefore, standard archive utilities are sufficient to extract its contents.

```bash
unzip -l AuroraPatientPortal.ipa
unzip -oq AuroraPatientPortal.ipa -d ipa
```

The extracted application bundle is located at:

```text
ipa/Payload/aurora-patient-portal.app/
```

Inside the bundle, several interesting files can be identified:

| File | Description |
|---|---|
| `Aurora Health Mobile` | ARM64 Mach-O executable |
| `Info.plist` | Binary property list containing application metadata |
| `aurora_meds.sqlite` | SQLite database containing medication-related information |
| `aurora_chest_xray.jpg` | Chest X-ray image asset |
| `embedded.mobileprovision` | iOS provisioning profile |
| `_CodeSignature/` | Application signing information |
| `Assets.car` | Compiled application UI assets |

The `Info.plist` file immediately stands out as a potential source of application configuration and embedded data.

## 3. Inspecting Info.plist

Since the plist is stored in binary format, Python's built-in `plistlib` module can parse it.

```python
import plistlib

with open("Info.plist", "rb") as f:
    data = plistlib.load(f)

print(data["AuroraPatientBootstrap"])
```

Among the standard application metadata keys, an unusual custom key appears:

```text
AuroraPatientBootstrap
```

Its value is a Base64-encoded string:

```text
eyJwYXRpZW50TmFtZSI6Ik1hcmEgVm9zcyIsImRhdGVPZkJpcnRoIjoiMTk4OC0wNC0xNyIsImNsaW5pYyI6Ildlc3RsYWtlIn0=
```

The key name suggests that it may contain initial patient information used by the application.

## 4. Decoding the Base64 Data

The encoded value can be decoded using the standard Linux `base64` utility.

```bash
echo 'eyJwYXRpZW50TmFtZSI6Ik1hcmEgVm9zcyIsImRhdGVPZkJpcnRoIjoiMTk4OC0wNC0xNyIsImNsaW5pYyI6Ildlc3RsYWtlIn0=' | base64 -d
```

The decoded output is:

```json
{
  "patientName": "Mara Voss",
  "dateOfBirth": "1988-04-17",
  "clinic": "Westlake"
}
```

This reveals the patient's full name and date of birth.

However, to establish that this information is actually consumed by the application rather than being unrelated leftover data, the Mach-O executable can also be inspected.

## 5. Confirming the Data Source Through Binary Analysis

Running `strings` against the application's Mach-O executable reveals several interesting references.

```bash
strings "Aurora Health Mobile" | grep -Ei 'PatientBootstrap|patientName|dateOfBirth|displayDateOfBirth|ageText'
```

The output contains references to:

- `PatientBootstrap` — a Swift structure associated with patient bootstrap data.
- `patientName` — the patient's name field.
- `dateOfBirth` — the date-of-birth field.
- `displayDateOfBirth` — a display-related accessor.
- `ageText` — an accessor related to patient age.
- `AuroraPatientBootstrap` — the custom Info.plist key.

These references establish a connection between the custom plist entry and the application's patient bootstrap model.

Therefore, the Base64-encoded value is not simply arbitrary metadata. It contains patient information that the application is designed to read.

## 6. Flag

The extracted patient information is:

- **Patient:** Mara Voss
- **Date of Birth:** 1988-04-17
- **Clinic:** Westlake

The required flag format is `deadface{firstname-lastname-YYYY-MM-DD}`.

```text
deadface{mara-voss-1988-04-17}
```

## 7. Root Cause and Security Impact

The root cause is the inclusion of patient-specific information directly inside the distributed iOS application bundle.

The application stores the patient's name, date of birth, and clinic in plaintext JSON encoded with Base64 inside `Info.plist`.

Base64 is an encoding mechanism, not encryption. Anyone with access to the IPA can extract the application bundle and decode the information without authentication, a jailbreak, or runtime debugging.

This creates a significant privacy risk:

- **Personally Identifiable Information (PII) Exposure:** Patient names and dates of birth are accessible to anyone who obtains the IPA.
- **Targeted Phishing:** Attackers can use authentic patient details to create convincing phishing messages.
- **Unauthorized Data Access:** Sensitive information is exposed outside the intended authenticated application session.

The exposure directly matches the challenge's phishing scenario, where attackers possess accurate patient-specific information.

## 8. Remediation

Several measures could prevent this type of information disclosure:

1. **Never embed patient-specific information in application bundles.** Avoid storing PII in `Info.plist`, source code, configuration files, or other distributed assets.

2. **Use authenticated API requests.** Retrieve patient records from a properly secured backend after the user authenticates.

3. **Secure locally cached information.** Use iOS Keychain and appropriate iOS Data Protection mechanisms for sensitive data that must be stored locally.

4. **Remove development and demo data.** Ensure test records and patient-specific information are excluded from production builds.

5. **Implement automated security checks.** Add CI/CD scanning to detect PII, credentials, and other sensitive information inside release artifacts.

6. **Audit bundled resources.** Inspect databases, images, configuration files, and other assets for accidental data exposure.

## 9. Additional Notes

The challenge description also contains:

```text
flag{1_l0v3_AI}
```

However, this does not match the required `deadface{...}` flag format and is not the answer to this challenge.

Additionally, the bundled `aurora_meds.sqlite` database and `aurora_chest_xray.jpg` image were not required to solve this particular task. They may contain information relevant to other challenges in the Aurora series.

## Conclusion

This challenge demonstrates how seemingly harmless application configuration files can expose sensitive patient information.

By unpacking the IPA, inspecting its `Info.plist`, decoding a Base64 value, and validating the associated Swift model references in the Mach-O executable, we identified the exposed patient's information entirely through static analysis.

**Key takeaway:** Never treat Base64 encoding or application-bundle obfuscation as a security mechanism for sensitive data.
