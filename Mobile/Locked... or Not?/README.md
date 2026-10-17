# Locked... or Not?
**Platform:** DEADFACE CTF  
**Category:** Mobile  
**Points:** 150  
**Author:** @G2Gh0st  

## Challenge Description

DEADFACE's junior developer wrote an Android application for storing secure notes. One of the notes is password-protected, but the password is unknown.

The objective is to analyze the Android application, bypass the password protection, and recover the hidden flag.

## Flag

```text
deadface{ez-mobile-to-start}
```

---

## Solution

### 1. Identify the APK

First, I checked the file type to confirm that the provided file was an Android application package.

```bash
file df-securenotes.apk
```

**Output:**
```text
Android package (APK), with AndroidManifest.xml, with APK Signing Block
```

An APK is essentially a ZIP archive containing the application's code, resources, and metadata.

### 2. Extract the APK

Since an APK is a ZIP archive, I extracted its contents using `unzip`.

```bash
mkdir apk
cd apk
unzip ../df-securenotes.apk
```

The extracted files included:

```text
AndroidManifest.xml
classes.dex
classes2.dex
resources.arsc
res/
META-INF/
```

While `classes.dex` mainly contained generated Android resource classes, `classes2.dex` contained the application's actual logic.

### 3. Analyze DEX Strings

Instead of immediately decompiling the application, I started with a quick static analysis using the `strings` command.

```bash
strings -n 4 classes2.dex
```

Among the output, I discovered several interesting strings:

- Package name: `com.deadface.securenoteslite`
- Activities: `MainActivity` and `NoteDetailActivity`
- UI messages: `Authentication Required`, `Access denied`, and `Unlock`
- Password prompt: `This note is protected. Enter the password to continue.`
- Decoy notes: `Grocery List` and `Meeting Notes`
- Interesting method: `getSecureBackupText`
- Interesting field: `apiKeyBackup`
- A Base64-encoded string located near these identifiers.

The suspicious Base64 string was:

```text
ZGVhZGZhY2V7ZXotbW9iaWxlLXRvLXN0YXJ0fQ==
```

The method and field names suggested that the application might contain a backup of the protected note.

### 4. Decode the Base64 String

The suspicious string appeared to be Base64-encoded. I decoded it using the following command:

```bash
echo 'ZGVhZGZhY2V7ZXotbW9iaWxlLXRvLXN0YXJ0fQ==' | base64 -d
```

**Output:**

```text
deadface{ez-mobile-to-start}
```

And there it was! The flag was recovered without entering the password or even running the application.

---

## Root Cause

The application stored sensitive information directly inside its APK as a Base64-encoded string.

Base64 is an encoding mechanism, not encryption. Anyone who can access the APK can extract and decode the string.

Although the application displayed a password prompt, the sensitive data was already present in the client-side code. Therefore, the UI-level password protection did not actually protect the underlying secret.

Interestingly, one of the decoy notes even contained the message:

> Do not store secrets in client-side code.

A pretty ironic warning!

## Tools Used

| Tool | Purpose |
|---|---|
| `file` | Identify the APK file |
| `unzip` | Extract APK contents |
| `strings` | Search for interesting strings in DEX files |
| `base64` | Decode the hidden flag |

## Key Takeaways

- **Never store secrets in client-side code:** APK files can be extracted and reverse-engineered.
- **Base64 is not encryption:** Encoding does not provide confidentiality.
- **UI protection is not data protection:** A password dialog cannot secure information that is already accessible inside the application.
- **Perform static analysis first:** Running `strings` against DEX files is a quick and effective first step when analyzing Android applications.
- **Use proper encryption:** Sensitive local data should be encrypted using securely managed keys, ideally derived from user credentials where appropriate. Server-side authorization should protect server-held secrets.

## Conclusion

This was a straightforward Android reverse-engineering challenge. By extracting the APK and analyzing its DEX strings, I discovered a Base64-encoded backup containing the flag.

No dynamic analysis or password bypass was necessary.

**Challenge solved!** 🚩
