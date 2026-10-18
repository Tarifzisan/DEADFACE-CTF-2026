# But A Wee Bit Reverse 

**Category:** Reverse Engineering  
**Flag:** `deadface{x0r_1s_not_encryption}`

## Overview

The challenge provides a binary named `re02.bin`. It is a 64-bit PIE ELF executable that asks for a password and prints the flag if the input is correct.

The challenge description hints that `strings` won't reveal much. That's because both the expected password and the flag are XOR-encoded and aren't stored as readable plaintext.

## 1. Reconnaissance

First, I checked the binary type:

```bash
file re02.bin
```

Output:

```text
ELF 64-bit LSB pie executable, x86-64, dynamically linked, not stripped
```

Since the binary is not stripped, its symbol table provides useful function and variable names.

The important symbols are:

- `main`
- `pw_enc` at offset `0x4010`
- `flag_enc` at offset `0x4020`

The imported functions also provide some clues about the program's behavior:

- `fgets` – reads user input.
- `strcspn` – removes the trailing newline.
- `strlen` – checks the input length.
- `putchar` – prints the decrypted flag character by character.

## 2. Static Analysis

I disassembled the `main` function using:

```bash
objdump -d -M intel re02.bin
```

The program's logic consists of two stages.

### Stage 1: Password Verification

The binary performs the following operations:

1. Reads input using `fgets(buf, 0x40, stdin)`.
2. Removes the trailing newline using `strcspn`.
3. Checks whether the input length is exactly 10 bytes.
4. Iterates through each character and verifies:

```c
(buf[i] ^ 0x42) == pw_enc[i]
```

This means each password character is XORed with the single-byte key `0x42` before being compared with the encoded password.

Because XOR is reversible, we can recover the original password using:

```python
password[i] = pw_enc[i] ^ 0x42
```

### Stage 2: Flag Decryption

If the password is correct, the program prints:

```text
Access granted!
```

It then decrypts the flag using another XOR operation.

The relevant logic is:

```c
flag[i] = flag_enc[i] ^ buf[i % 10];
```

Interestingly, the password itself acts as the repeating XOR key for decrypting the flag.

The loop runs 31 times (`0x1f`), which corresponds to the length of the encoded flag.

## 3. Extracting the Encoded Data

The encoded password and flag are stored in the binary's `.data` section.

The relevant raw bytes are:

**Encoded Password (`pw_enc`, offset `0x4010`):**

```text
31 37 2c 31 2a 2b 2c 27 76 70
```

**Encoded Flag (`flag_enc`, offset `0x4020`):**

```text
17 10 0f 17 0e 08 0d 00 4f 4a 43 07 31 42 1b 36
00 0a 40 6d 16 1b 0d 01 11 19 1a 0c 5b 5c 0e
```

## 4. Writing the Solver

With both encoded byte sequences extracted, I wrote a small Python script to recover the password and decrypt the flag.

```python
pw_enc = bytes.fromhex("3137 2c31 2a2b 2c27 7670".replace(" ", ""))

flag_enc = bytes.fromhex(
    "17100f170e080d00"
    "4f4a430731421b36"
    "000a406d161b0d01"
    "11191a0c5b5c0e"
)

# Recover the password
pw = bytes(b ^ 0x42 for b in pw_enc)

# Decrypt the flag using the repeating password as the XOR key
flag = bytes(
    flag_enc[i] ^ pw[i % len(pw)]
    for i in range(len(flag_enc))
)

print("Password:", pw.decode())
print("Flag:", flag.decode())
```

**Output:**

```text
Password: sunshine42
Flag: deadface{x0r_1s_not_encryption}
```

## 5. Final Answer

**Password:** `sunshine42`

**Flag:**

```text
deadface{x0r_1s_not_encryption}
```

## 6. Takeaways

- XOR encoding can hide strings from basic tools like `strings`, but it does not provide meaningful encryption when the key or decryption logic is embedded in the binary.
- Repeating-key XOR is reversible. Knowing the encoded data and the key is enough to recover the original plaintext.
- Known-plaintext attacks can also help recover XOR keys. For example, XORing the expected flag prefix `deadface{` with the corresponding encoded bytes reveals the first characters of the repeating key.
- Static analysis with `objdump` and a short Python script was sufficient to solve this challenge. No debugger or dynamic execution was necessary.

**Challenge solved!**
