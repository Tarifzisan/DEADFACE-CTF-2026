# Cereal Killer 02

**Challenge:** Cereal Killer 02  
**Author:** @TheZeal0t  
**Category:** Reverse Engineering  
**Points:** 150  

**Flag:** `deadface{N0rdics-R3ally-like-Frank3nb3rry-cuz-bro-is-tall-like-they-are!}`

## Challenge Overview

The challenge provides a Linux binary that XORs two embedded 256-byte tables and searches the resulting buffer for the marker `badf00d`.

At first, the program reports that the key has been tampered with. However, the error messages contain an important hint: **"TIME is REVERSIBLE, but the KEY is NOT..."**

The actual trick is that the key table is stored in reverse order. Reversing the key before performing the XOR operation reveals a 16×16 text grid containing the flag.

## 1. Getting the Files

The provided archive, `re03-ck02.7z`, is protected with AES-256 encryption and an encrypted header, meaning even the filenames are hidden.

The archive password is:

```text
KlippyAI
```

After extraction, we get three files:

- `re03-lin-x86.bin` — Linux challenge binary (32-bit ELF)
- `re03-win-x86.exe` — Windows challenge binary
- `KlippyAI-trained-on-CK02.exe` — Packed bonus program

For this analysis, I focused on the Linux ELF binary and did not execute the bonus program.

## 2. Initial Triage

First, I checked the binary type using `file` and inspected its printable strings.

The binary is a **32-bit PIE ELF executable** and is not stripped.

Running `strings` reveals an ASCII-art banner and two particularly interesting error messages:

```text
The key has been tampered with. It does not correctly decipher the flag!

TIME is REVERSIBLE, but the KEY is NOT... Go Back! GO BACK!!!
```

The second message immediately suggests that reversing something might be necessary.

## 3. Analysing `main`

I disassembled the `main` function to understand how the program processes the embedded data.

The relevant operations are:

1. Two 256-byte tables are copied onto the stack:
   - Ciphertext table `A`, located at file offset `0x2a20`.
   - Key table `B`, located at file offset `0x2b24`.

2. A loop performs a byte-by-byte XOR operation:

   ```c
   out[i] = A[i] ^ B[i];
   ```

   This operation runs for all 256 bytes.

3. The program constructs the 7-byte marker `badf00d` on the stack.

4. A custom `memmem(0x100, out, 7, needle)` function searches the output buffer for this marker.

5. If the marker is found, the program prints the success message and the resulting buffer. Otherwise, it displays the tampered-key error.

I also inspected the custom `memmem` implementation. It performs a straightforward linear search, so the search function itself is not responsible for the failure.

## 4. Finding the Reversal Trick

Initially, performing the XOR operation directly produces random-looking output without the expected `badf00d` marker.

However, examining the output reveals an approximate mirror-like pattern. Combined with the hint in the error message, this suggests that the key might be stored in reverse order.

I tested the reversal combinations and found that reversing the key table produces the expected marker:

```python
out = bytes(a ^ b for a, b in zip(A, B[::-1]))
```

This successfully reveals `badf00d` in the resulting buffer.

The important detail is that the ciphertext remains unchanged; only the key table needs to be reversed before XOR.

## 5. Examining the Decrypted Grid

The decrypted 256-byte buffer forms a 16×16 text grid:

```text
****************
* f e e d . . .*
* f a c e . . .*
* d e a d b eef*
* d e a d . . .*
* f a c e { . .*
* N 0 r d i c s*
*-R3ally-like- *
*Frank3nb3rry- *
* cuz-bro-is- .*
* tall-like-. .*
* they-are!} . *
* badf00d......*
* d a 7e4107 . *
****************
```

Several recognizable strings appear throughout the grid:

- `feed`
- `face`
- `deadbeef`
- `badf00d`
- `da7e4107`

These are decoys.

The actual flag begins with `face{` in the fifth data row, continuing the `dead` prefix from the row immediately above it.

Following the flag characters through the subsequent rows reveals the complete message.

## 6. Reconstructing the Flag

The challenge instructions specify that the flag contains no spaces, periods, or asterisks.

Removing those characters and joining the relevant rows, starting from the `deadface{` prefix, gives:

```text
deadface{N0rdics-R3ally-like-Frank3nb3rry-cuz-bro-is-tall-like-they-are!}
```

One important detail: the `0` in `N0rdics` is the digit zero, not the letter `O`.

## Takeaways

- **Read error messages carefully:** Sometimes a seemingly ordinary error message contains the key to solving the challenge.
- **Understand the transformation:** When an XOR operation produces unexpected output, investigate the arrangement of its operands.
- **Look for structural patterns:** A mirror-like pattern in decrypted output can indicate reversed data.
- **Distinguish real data from decoys:** Recognizable hexadecimal strings and flag-like fragments do not necessarily belong to the actual flag.

The key to solving this challenge was recognizing that the embedded key table was reversed and correcting its order before performing the XOR operation.
