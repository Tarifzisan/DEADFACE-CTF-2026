# Cereal Killer 01 

**Category:** Reverse Engineering  
**Binary:** `re01-lin-x86.bin`  
**Architecture:** 32-bit ELF (i386), PIE, dynamically linked, not stripped  
**Flag:** `deadface{You-Vill-Eat-Ze-Bugz-Und-You-Vill-Be-Happy!!!}`

## 1. Challenge Overview

The challenge provides a Linux ELF binary that prompts the user for a password. If the password is correct, the program decrypts and displays a hidden message.

The challenge description revolves around alien "Insectoids" and their favorite terrestrial breakfast cereal. This narrative provides an important hint: the password is likely the name of a cereal brand.

## 2. Initial Binary Analysis

First, I identified the binary's architecture and properties using `file`.

```bash
file re01-lin-x86.bin
```

Output:

```text
ELF 32-bit LSB pie executable, Intel 80386, dynamically
linked, interpreter /lib/ld-linux.so.2, not stripped
```

Since the binary is not stripped, its function symbols are preserved. I used `nm` to inspect the available functions:

```bash
nm re01-lin-x86.bin | grep ' T '
```

Several interesting functions immediately stood out:

- `SHA1Init`, `SHA1Update`, `SHA1Final`, `SHA1` — SHA-1 implementation.
- `KSA`, `PRGA`, `RC4`, `swap` — RC4 implementation.
- `base64_decode` — Custom Base64 decoder.
- `main` — Main program logic.

This suggested that SHA-1 was involved in password verification and RC4 was used to decrypt the hidden payload.

## 3. Analyzing the Main Function

I disassembled the binary using:

```bash
objdump -d -M intel re01-lin-x86.bin
```

By following the control flow in `main`, I identified the following operations:

1. Two embedded Base64-encoded blobs are decoded during initialization:
   - The target SHA-1 hash.
   - A separate set of bytes used later in the program.
2. The program prints its banner and challenge-related messages.
3. `getline()` reads the password from standard input and removes the trailing newline.
4. The program calculates the SHA-1 hash of the supplied password.
5. Each hash byte is converted into a two-character hexadecimal representation using `sprintf()`.
6. `strcmp()` compares the resulting 40-character hexadecimal string against the decoded target hash.

If the hashes do not match, the program prints `ACCESS DENIED` and exits.

If they match, the program decodes another Base64 blob and decrypts it using RC4, with the original password as the key. Finally, `fwrite()` outputs the decrypted plaintext.

**Important observation:** The password serves two purposes: it must match the expected SHA-1 digest, and it is also used directly as the RC4 key. Simply bypassing the password comparison would not reveal the original plaintext.

## 4. Reverse-Engineering the Custom Base64 Alphabet

One interesting part of this challenge is that the Base64 decoder does not use the standard Base64 alphabet.

By examining the decoder's GOT-relative addressing and following the references through `get_pc_thunk.bp`, I traced the lookup table to `.rodata` at offset `0x4920`.

The custom alphabet is:

```text
ZKj9n+yf0wDVX1s/5YbdxSo=ILaUpPBCHg8uvNO4klm6iJGhQ7eFrWczAMEq3RTt
```

An important detail is that `=` is an ordinary character in this alphabet, located at index 24. It is not treated as a padding character.

I reconstructed the decoder in Python:

```python
alpha = "ZKj9n+yf0wDVX1s/5YbdxSo=ILaUpPBCHg8uvNO4klm6iJGhQ7eFrWczAMEq3RTt"

def dec(s):
    out = bytearray()

    for i in range(0, len(s), 4):
        chunk = s[i:i+4]
        v = [alpha.find(c) for c in chunk]

        out.append(((v[0] << 2) | (v[1] >> 4)) & 255)

        if len(chunk) > 2 and v[2] != -1:
            out.append(((v[1] << 4) | (v[2] >> 2)) & 255)

            if len(chunk) > 3 and v[3] != -1:
                out.append(((v[2] << 6) | v[3]) & 255)

    return bytes(out)
```

Using this custom decoder, I recovered two important pieces of data:

**Target SHA-1 digest:**

```text
68158678a3f9d76a3d1ca12d20b2d6af9bd3f290
```

**Encrypted payload:** 59 bytes of RC4 ciphertext.

## 5. Cracking the Password

With the SHA-1 digest recovered, the next step was identifying the original password.

The challenge description strongly suggested a cereal brand. Instead of blindly brute-forcing every possible combination, I used a targeted dictionary attack against common cereal names and their formatting variations.

The following Python snippet verifies the discovered password:

```python
import hashlib

hashlib.sha1(b"cookiecrisp").hexdigest()

# 68158678a3f9d76a3d1ca12d20b2d6af9bd3f290
```

The calculated digest matches the target hash exactly.

**Recovered Password:** `cookiecrisp`

This corresponds to the cereal brand **Cookie Crisp**.

## 6. Decrypting the RC4 Payload

Since the original password is also used as the RC4 key, I implemented the Key Scheduling Algorithm (KSA) and Pseudo-Random Generation Algorithm (PRGA) in Python.

```python
def ksa(key):
    S = list(range(256))
    j = 0

    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]

    return S


def prga(S, n):
    i = j = 0
    out = bytearray()

    for _ in range(n):
        i = (i + 1) % 256
        j = (j + S[i]) % 256

        S[i], S[j] = S[j], S[i]

        out.append(S[(S[i] + S[j]) % 256])

    return out


key = b"cookiecrisp"

S = ksa(key)

plaintext = bytes(
    a ^ b
    for a, b in zip(ciphertext, prga(S, len(ciphertext)))
)

print(plaintext)
```

The decrypted output was:

```text
https://cerealkiller.lyttonlabs.org/unwanted-guests/ick.png
```

This result also matches the output produced by the original binary when supplied with the correct password, confirming that the password and decryption process are correct.

## 7. Retrieving the Final Flag

The decrypted URL points to an image:

```text
https://cerealkiller.lyttonlabs.org/unwanted-guests/ick.png
```

The image contains a fictional Lytton Labs staff photograph featuring:

**Dr. Charles K. Gesch-ICK-ter, Director, Lytton Labs**

The actual flag is printed directly on the image.

```text
deadface{You-Vill-Eat-Ze-Bugz-Und-You-Vill-Be-Happy!!!}
```

One final detail worth mentioning: the `deadface{...}` string displayed in the binary's password prompt or challenge description is a decoy, not the actual flag.

## 8. Final Flag

```text
deadface{You-Vill-Eat-Ze-Bugz-Und-You-Vill-Be-Happy!!!}
```

## 9. Tools and Techniques

| Tool / Technique | Purpose |
|---|---|
| `file` | Identify binary architecture and properties |
| `nm` | Enumerate available function symbols |
| `objdump` | Disassemble and inspect program logic |
| SHA-1 | Understand and reproduce password verification |
| Custom Base64 decoder | Recover embedded binary data |
| Python | Implement decoding, hashing, and decryption |
| RC4 (KSA + PRGA) | Decrypt the hidden URL |
| Dictionary attack | Recover the cereal-based password |

### Key Takeaways

- Preserved symbol information can significantly simplify reverse engineering.
- Custom encoding implementations should never be assumed to follow standard Base64 behavior.
- A known hash enables targeted dictionary attacks when the challenge provides useful hints.
- Understanding the complete execution flow is more effective than simply patching a password check.
- Combining static analysis with small Python scripts makes it easier to reproduce and validate a binary's behavior.
