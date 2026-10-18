# Hard Kept Secrets — DEADFACE CTF Write-up

**Category:** Mobile / Android Reverse Engineering  
**Points:** 20  
**Author:** @syyntax  
**Event:** DEADFACE CTF  
**File:** `app-release.apk`

## Challenge Description

De Monne Financial has experienced several security incidents involving its latest Android application. DEADFACE appears to have gained access to the company's API.

Before assuming that a password was compromised, we need to investigate the Android APK for hardcoded secrets and recover the **internal API token**.

**Flag Format:** `deadface{...}`

## Flag

```text
deadface{zqJRlVwBA9Xm2lE5EPjgbj/4rOxlkYZgDtkdP7jV80rZgLFrlnOEn+VAu31YKQrR}
```

## Solution

### 1. APK Initial Analysis

An Android APK is essentially a ZIP archive. We can inspect its contents using `unzip`.

```bash
unzip -l app-release.apk
```

Extract the APK contents:

```bash
unzip -o app-release.apk -d apk/
```

The APK contains two DEX files:

- `classes.dex`
- `classes2.dex`

There are no native libraries, and no relevant secrets were found in the assets directory. This suggests that the application's important logic is primarily implemented in its DEX bytecode.

### 2. Searching for Interesting Strings

The first step was to inspect the DEX files for potentially sensitive strings.

```bash
strings apk/classes.dex | grep -Ei 'deadface|token|secret|bearer'
strings apk/classes2.dex | grep -Ei 'deadface|token|secret|bearer'
```

The search revealed two flag-shaped strings:

| String | Location / Context | Analysis |
|---|---|---|
| `deadface{flaG_ManIfesT3d}` | Diagnostic string | Decoy, unrelated to API authentication |
| `deadface{Ur_N0t_A110wEd_h3r3}` | `SecureVaultActivity.VAULT_CONTENT` | Decoy displayed on a restricted vault screen |

Although these strings looked interesting, neither represented the requested internal API token.

The search also revealed several useful identifiers:

- `INTERNAL_API_TOKEN`
- `InternalTokenInterceptor`
- `AuthInterceptor`
- `BuildConfig`

These identifiers provided a starting point for investigating the application's authentication logic.

### 3. Analyzing DEX Static Values

Since JADX and APKTool were unavailable in my environment, I wrote a small custom Python DEX parser to inspect the application's compiled bytecode.

The parser performed the following operations:

1. Read the DEX string table.
2. Parsed the `class_defs` section.
3. Traversed `class_data_item` structures.
4. Decoded static field values from `static_values`.

This allowed me to recover compile-time constants, including hardcoded `static final String` values.

After filtering classes belonging to the `com.demonne` package, I identified several interesting findings.

**BuildConfig**

```text
API_BASE_URL = http://dmf.dctf.io:8081
BUILD_ENV    = production
VERSION_NAME = 2.4.1
```

**LoginActivity**

The application contained hardcoded internal credentials:

```text
Username: audit@demonne-financial.local
Password: demonnefinancial
```

**ApiClient**

Most importantly, the `api.ApiClient` class contained the following constant:

```text
INTERNAL_API_TOKEN =
zqJRlVwBA9Xm2lE5EPjgbj/4rOxlkYZgDtkdP7jV80rZgLFrlnOEn+VAu31YKQrR
```

This appeared to be the secret we were looking for.

### 4. Verifying the Internal API Token

Before submitting the value, I wanted to verify whether it was actually the API token or an encrypted value requiring additional processing.

The string is 64 characters long and Base64-decodes into 48 bytes. However, its appearance alone does not establish whether it is encrypted.

I therefore inspected the relevant DEX instructions and followed the token's usage.

**ApiClient.getInstance()**

The method loads the token using the `const-string/jumbo` instruction and passes it directly to the `InternalTokenInterceptor` constructor.

**InternalTokenInterceptor.intercept()**

The interceptor uses the supplied value to set the following HTTP request header:

```http
X-Internal-Token: zqJRlVwBA9Xm2lE5EPjgbj/4rOxlkYZgDtkdP7jV80rZgLFrlnOEn+VAu31YKQrR
```

The value is passed directly into the request header without any decryption, decoding, or key derivation.

This confirms that the hardcoded string is the actual internal API token.

## Final Flag

```text
deadface{zqJRlVwBA9Xm2lE5EPjgbj/4rOxlkYZgDtkdP7jV80rZgLFrlnOEn+VAu31YKQrR}
```

## Security Findings

This challenge demonstrates several security weaknesses in the De Monne Financial Android application.

**1. Hardcoded API Token**

The internal API token is embedded directly in the APK. Anyone with access to the application can extract it through static analysis.

**2. Cleartext HTTP Communication**

The API endpoint uses HTTP instead of HTTPS:

```text
http://dmf.dctf.io:8081
```

Consequently, authentication tokens transmitted over the network may be exposed to interception.

**3. Hardcoded Credentials**

The `LoginActivity` class contains internal credentials in plaintext, creating another potential attack vector.

### Recommended Remediation

- Immediately rotate the exposed API token and internal credentials.
- Avoid embedding long-lived shared secrets in client-side applications.
- Implement per-user authentication with short-lived access tokens.
- Enforce HTTPS/TLS for all API communications.
- Consider application integrity verification, such as Google Play Integrity, as an additional security layer.

## Tools Used

- `unzip` — APK extraction and inspection
- `strings` — Extracting readable strings from DEX files
- `grep` — Searching for interesting identifiers and secrets
- Python — Custom DEX parser
- DEX bytecode analysis — Tracing token initialization and HTTP header usage

## Key Learning

**Anything shipped inside a client-side application should be considered accessible to the user.**

Even when an application does not expose its secrets through the interface, static analysis can reveal hardcoded credentials, API tokens, and other sensitive information.

In this challenge, tracing a hardcoded constant from `ApiClient` to `InternalTokenInterceptor` was the key to distinguishing the real API token from the decoy flags.
