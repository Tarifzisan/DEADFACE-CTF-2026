# DEADFACE CTF – Headshot (Steg, 10 Points)

## Challenge Description

DEADFACE tampered with one of several Aurora Health corporate headshots. The objective was to identify the modified image and recover the hidden flag.

**Category:** Steganography  
**Difficulty:** Easy  
**Points:** 10

**Flag:**
```text
deadface{b00m_h34Dsh0TTT}
```

---

## 1. Initial File Analysis

After extracting the provided ZIP archive, I found four PNG files:

- `headshot001.png`
- `headshot002.png`
- `headshot003.png`
- `headshot003.png_original`

First, I checked the file types, SHA1 hashes, and metadata to understand the structure of the images.

```bash
file *
sha1sum *
exiftool headshot00*.png
```

All images had the same basic properties:

- Resolution: 512 × 512
- Color depth: 8-bit
- Color type: Indexed palette
- No useful EXIF metadata

However, one interesting detail stood out: `headshot003.png_original` appeared to be the original, unmodified version of `headshot003.png`.

Their file sizes were slightly different:

| File | Size (Bytes) |
|---|---:|
| headshot003.png | 182,911 |
| headshot003.png_original | 182,866 |

The difference was exactly **45 bytes**, suggesting that something had been added to the modified image.

## 2. Extracting the Hidden Flag

Instead of immediately trying pixel-level steganography techniques, I started with a simple string search.

```bash
strings -n 8 headshot003.png | grep -i deadface
```

Output:

```text
deadface{b00m_h34Dsh0TTT}
```

And just like that, the flag was recovered!

The flag was not present in `headshot001.png`, `headshot002.png`, or the original copy of `headshot003.png`.

This confirmed that `headshot003.png` was the tampered image.

## 3. Investigating the PNG Structure

To understand how the flag was hidden, I compared the PNG chunk structures of the modified image and its original copy.

PNG files consist of several chunks, each following this structure:

```text
Length (4 bytes) | Chunk Type (4 bytes) | Data | CRC (4 bytes)
```

A comparison revealed an additional `tEXt` chunk in the modified image.

| Offset | Chunk | Modified Image | Original Image |
|---|---|---|---|
| 8 | IHDR | Present | Present |
| 33 | PLTE | Present | Present |
| 813 | tRNS | Present | Present |
| 1081 | pHYs | Present | Present |
| 1102 | tEXt | Added | Missing |
| 1147 | IDAT | Present | Present |
| End | IEND | Present | Present |

The additional chunk contained:

```text
Comment\0deadface{b00m_h34Dsh0TTT}
```

This means the flag was stored inside a PNG text metadata chunk using the keyword `Comment`.

The 45-byte difference can be explained as follows:

- Chunk framing: 12 bytes (length, type, and CRC)
- Chunk data: 33 bytes (`Comment` + null terminator + flag)

Interestingly, the IDAT image data was identical in both files (181,740 bytes). Therefore, the actual pixel data had not been modified.

The image looked completely normal because the hidden information was stored in metadata rather than in the image pixels.

## 4. Identifying the Tampered Image

Based on the comparison, `headshot003.png` was clearly the modified image.

It was the woman wearing a navy blazer.

The presence of the extra `tEXt` chunk and the recovered flag confirmed that this was the targeted corporate headshot.

## 5. Key Takeaways

- PNG files can contain hidden information in metadata chunks such as `tEXt`.
- `strings` is a useful first step when investigating suspicious files.
- Comparing an image against a known-good original can reveal subtle modifications.
- Pixel-level steganography tools such as `zsteg` and StegSolve are not always necessary.
- Tools like `pngcheck` and `binwalk` can help identify unusual PNG chunks.

**Final Flag:**

```text
deadface{b00m_h34Dsh0TTT}
```

**Conclusion:** The challenge was solved by identifying an extra PNG `tEXt` metadata chunk in `headshot003.png`. No pixel manipulation or traditional LSB steganography was involved.
