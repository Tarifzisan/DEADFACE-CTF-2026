# Exfiltration:

**Challenge:** Exfiltration (75 Points)  
**Category:** Steganography  
**Author:** @G2Gh0st  
**Event:** DEADFACE CTF  
**Files:** `exfiltration.zip` (20 JPEG images)

## Challenge Description

DEADFACE frequently uses unconventional methods to exchange information and avoid detection. Analysts recovered a collection of images from a compromised communications channel. Although the images appeared unrelated, they suspected the collection contained a hidden message.

The objective was to recover the hidden message and identify what DEADFACE was trying to communicate.

## Solution

### 1. Extracting and Inspecting the Images

After downloading and extracting `exfiltration.zip`, I found 20 JPEG images named `image_01.jpg` through `image_20.jpg`.

First, I inspected the file types and image dimensions:

```bash
file *.jpg
```

Most images had dimensions of `307x256`, while a few were `308x256`.

Next, I examined the EXIF metadata of an image:

```bash
exiftool image_01.jpg
```

The metadata contained an interesting field:

```text
Image Unique ID : IMG_6365
Date/Time Original : 2026:06:01 14:13:09
```

This suggested that the `Image Unique ID` field might contain hidden information.

### 2. Extracting Metadata from All Images

I extracted the filename, file size, unique ID, original timestamp, and image dimensions from all 20 images:

```bash
exiftool -FileName -FileSize -ImageUniqueID \
-DateTimeOriginal -ImageWidth -ImageHeight *.jpg
```

The `Image Unique ID` values looked unusual. Some contained four hexadecimal characters, while others contained only two.

For example:

```text
IMG_6465
IMG_61
IMG_6466
IMG_61
IMG_6365
IMG_7b
```

This suggested that the hexadecimal portions might represent ASCII characters.

### 3. Discovering the Correct Order

However, the images were not arranged in the correct sequence by their filenames.

The important clue was the `Date/Time Original` EXIF field.

I extracted the timestamps and unique IDs, then sorted the results chronologically:

```bash
exiftool -T -DateTimeOriginal -ImageUniqueID *.jpg | sort
```

This revealed the correct order in which the hidden message fragments needed to be assembled.

### 4. Decoding the Hexadecimal Values

After sorting, I removed the `IMG_` prefix from each unique ID.

The remaining hexadecimal values represented ASCII characters.

For example:

| Hexadecimal | ASCII |
|---|---|
| `6465` | `de` |
| `61` | `a` |
| `6466` | `df` |
| `61` | `a` |
| `6365` | `ce` |
| `7b` | `{` |
| `7469` | `ti` |
| `6d` | `m` |
| `696e` | `in` |
| `67` | `g` |
| `2d69` | `-i` |
| `73` | `s` |
| `2d65` | `-e` |
| `76` | `v` |
| `6572` | `er` |
| `79` | `y` |
| `7468` | `th` |
| `69` | `i` |
| `6e67` | `ng` |
| `7d` | `}` |

### 5. Automating the Extraction

Instead of manually decoding every hexadecimal fragment, I used a simple command pipeline:

```bash
exiftool -T -DateTimeOriginal -ImageUniqueID *.jpg \
| sort \
| awk '{print $3}' \
| sed 's/^IMG_//' \
| xxd -r -p

echo
```

**Output:**

```text
deadface{timing-is-everything}
```

## Flag

```text
deadface{timing-is-everything}
```

## Key Takeaways

- EXIF metadata can contain information beyond ordinary camera details.
- The `Image Unique ID` field can be abused to store hexadecimal-encoded data.
- Metadata timestamps can reveal the correct sequence of scattered information.
- Multiple seemingly unrelated images may collectively contain a hidden message.

**The key to this challenge was understanding that timing is everything!**
