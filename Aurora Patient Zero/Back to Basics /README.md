# Back to Basics 

**Challenge:** Back to Basics  
**Category:** Mobile App / Static Analysis  
**Points:** 15  
**CTF:** DEADFACE CTF 2026

**Flag:** `deadface{gw-westlake-int-03}`

## 1. Challenge Objective

The Aurora Patient Portal application contains a hidden diagnostic page that is not accessible through its normal navigation.

The objective is to identify the name of the gateway server the application uses to communicate with Aurora's internal servers.

Since the previous challenges already provided the application IPA and its associated resources, we can continue investigating the same extracted application bundle.

No emulator or debugger is required. Static analysis of the Mach-O executable is sufficient.

## 2. Discovering the Hidden Diagnostic Screen

The first step is to inspect the application's main Mach-O executable:

```bash
strings "Aurora Health Mobile" | grep -Ei 'DiagnosticScreen|DiagnosticRow|DiagnosticItem|_isShowingDiagnosticScreen'
```

The output reveals several interesting references:

- `DiagnosticScreen`
- `DiagnosticRow`
- `DiagnosticItem`
- `_isShowingDiagnosticScreen`

These names suggest that the application contains a dedicated diagnostic interface.

The `_isShowingDiagnosticScreen` state variable is particularly interesting because it indicates that the screen has its own visibility state.

Although the diagnostic interface is not included in the normal application navigation, its implementation and associated strings remain embedded in the executable.

## 3. Extracting Diagnostic Information

After identifying the diagnostic-related components, the next step is to inspect the surrounding string table.

```bash
strings "Aurora Health Mobile" | grep -Ei 'Aurora Diagnostics|Gateway|Westlake|Patient Cache|Imaging Link|Medication Cache|Last Gateway Check'
```

The extracted strings reveal several diagnostic entries:

| Diagnostic Row | Extracted Value |
|---|---|
| Screen Title | Aurora Diagnostics |
| Description | Internal clinic sync status |
| Gateway | `gw-westlake-int-03` |
| Patient Cache Realm | `patient-summary-cache-v2` |
| Imaging Link Mode | local bundled asset |
| Medication Cache | offline lookup enabled |
| Last Gateway Check | 2026-05-30 09:42 UTC |

The most important entry is:

```text
Gateway: gw-westlake-int-03
```

This appears to be the internal gateway hostname used by the application.

## 4. Eliminating Potential Decoys

To make sure there are no competing gateway candidates, search the extracted application bundle for other gateway-style identifiers.

```bash
grep -RIna 'gw-' .
```

The search returns one relevant match, located in the Mach-O executable.

Additionally, the hostname contains `westlake`, which matches the clinic name discovered in the first challenge:

```json
{
  "patientName": "Mara Voss",
  "dateOfBirth": "1988-04-17",
  "clinic": "Westlake"
}
```

This provides contextual consistency between the diagnostic information and the previously identified patient record.

Therefore, `gw-westlake-int-03` is the strongest candidate for the requested gateway name.

## 5. Flag

The extracted gateway hostname is:

```text
gw-westlake-int-03
```

Using the standard DEADFACE flag wrapper:

```text
deadface{gw-westlake-int-03}
```

## 6. Root Cause and Security Impact

The application exposes internal infrastructure information through hardcoded diagnostic strings in its production Mach-O executable.

Although the diagnostic page is hidden from normal application navigation, hiding a screen does not prevent someone from examining the compiled binary.

Anyone who obtains the IPA can use basic static analysis tools such as `strings` to discover:

- Internal gateway hostnames.
- Diagnostic interface information.
- Patient cache configuration names.
- Offline medication lookup details.
- Internal synchronization-related metadata.

Such information can help attackers understand the application's internal architecture and identify infrastructure components for further investigation.

However, hostname disclosure alone does not establish that the gateway is externally reachable or vulnerable.

## 7. Remediation

1. **Remove internal diagnostics from release builds:** Exclude development-only diagnostic screens and related strings from production binaries.

2. **Use build configurations:** Keep internal debugging functionality behind development-only build flags.

3. **Avoid hardcoded infrastructure details:** Retrieve environment-specific gateway configuration securely at runtime.

4. **Apply proper access controls:** Ensure diagnostic functionality is protected by appropriate authentication and authorization.

5. **Review compiled binaries:** Include static analysis of release Mach-O executables in the CI/CD security pipeline.

6. **Monitor infrastructure access:** Review gateway access logs and investigate unexpected connection attempts to internal infrastructure.

## Conclusion

This challenge demonstrates that hiding an application's diagnostic interface does not protect the information embedded within its compiled executable.

By inspecting the Mach-O binary, identifying diagnostic-related symbols, and extracting the associated string values, we discovered the gateway hostname:

`gw-westlake-int-03`

**Key takeaway:** Hidden application functionality is not equivalent to protected functionality. Anything embedded in a distributed binary should be treated as potentially accessible to an attacker.
