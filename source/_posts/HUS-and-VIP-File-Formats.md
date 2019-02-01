---
title: HUS and VIP File Formats
date: 2019-01-25 18:49:03
tags:
---

[Source](http://www.jasonweiler.com/HUSandVIPFileFormatInfo.html "Permalink to HUS and VIP Embroidery File Formats")

# HUS and VIP Embroidery File Formats

## File Header Information (Same for HUS & VIP)

| Offset | Size      | Description                                                             |
| ------ | --------- | ----------------------------------------------------------------------- |
| `0x00` | `2 bytes` | HUS: Always `0x5B 0xAF`. VIP: Always `0x5D 0xFC`                        |
| `0x02` | `1 byte`  | HUS: Must be `0xC8`. VIP: Must be `0x90`                                |
| `0x03` | `1 byte`  | HUS: Always `0x00`. VIP: Always `0x01`                                  |
| `0x04` | `4 bytes` | `NumberOfStitches` - The total number of stitches defined in this file. |
| `0x08` | `4 bytes` | `NumberOfColors` - The total number of colors used in this pattern.     |
| `0x0C` | `2 bytes` | +X hoop-size offset as a positive number (same as +X from DST)          |
| `0x0E` | `2 bytes` | +Y hoop-size offset as a positive number (same as -Y from DST)          |
| `0x10` | `2 bytes` | -X hoop-size offset as a negative number (same as -X from DST)          |
| `0x12` | `2 bytes` | -Y hoop-size offset as a negative number (same as +Y from DST)          |
| `0x14` | `4 bytes` | Absolute file-offset to SECTION 1 (contains stitch attributes)          |
| `0x18` | `4 bytes` | Absolute file-offset to SECTION 2 (contains x-coordinates)              |
| `0x1C` | `4 bytes` | Absolute file-offset to SECTION 3 (contains y-coordinates)              |
| `0x20` | `8 bytes` | String identifier (I've never seen it used)                             |
| `0x28` | `2 bytes` | *Unknown*                                                               |

### HUS Color list information (HUS Only)

| Offset | Size                     | Description   |
| ------ | ------------------------ |-------------- |
| `0x2A` | `2*NumberOfColors bytes` | Color indices |

HUS files use a list of 2-byte color indices to express a progression of colors used in the pattern. This list starts immediately following the header information (offset `0x2A`) and is always `2*NumberOfColors` in length. The colors are applied to the pattern starting with the first one, and moving to the next whenever a color-change stitch is encountered. It isn't clear if there is an accepted set of colors and associated indices, but we've experimentally derived the following HUS color table:

| Value  | Colour           |
| ------ | ---------------- |
| `0x00` | Black            |
| `0x01` | Blue             |
| `0x02` | Light Green      |
| `0x03` | Red              |
| `0x04` | Purple           |
| `0x05` | Yellow           |
| `0x06` | Gray             |
| `0x07` | Light Blue       |
| `0x08` | Green            |
| `0x09` | Orange           |
| `0x0A` | Pink             |
| `0x0B` | Brown            |
| `0x0C` | White            |
| `0x0D` | Dark Blue        |
| `0x0E` | Dark Green       |
| `0x0F` | Dark Red         |
| `0x10` | Light Red        |
| `0x11` | Dark Purple      |
| `0x12` | Light Purple     |
| `0x13` | Dark Yellow      |
| `0x14` | Light Yellow     |
| `0x15` | Dark Gray        |
| `0x16` | Light Gray       |
| `0x17` | Dark Orange      |
| `0x18` | Light Orange     |
| `0x19` | Dark Pink        |
| `0x1A` | Light Pink       |
| `0x1B` | Dark Brown       |
| `0x1C` | Light Brown      |


### VIP Color table information (VIP Only)

| Offset | Size                   | Description |
| ------ | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0x2A` | 4 bytes                | *Unknown - Values don't appear to be used. Sometimes the value is zero. But when it's non-zero, it is roughly proportional to the number of colors. |
| `0x2E` | 4*NumberOfColors bytes | Encoded color information                                                                                                                           |

VIP files are more advanced than HUS files in that they actually contain the real 24-bit colors that were intended by the pattern author. The color information is stored as RGB-0 quadruplets, but they're also encoded with a simple double-XOR scheme. For example:

Encoded bytes:

    	48 70 DD B2  77 07 05 C3  48 0E D6 7A  70 E2 0E 40
    	4B 0A FA 1E  C5 64 DB B3  EE 36 C9 7D

becomes:

    	66 BA 49 00  FD D9 DE 00  F0 F0 F0 00  F7 38 66 00
    	7D 6F 00 00  FE BA 35 00  13 4A 46 00

The decoding algorithm is actually quite simple:

    	BYTE prevByte = 0
    	BYTE tmpByte = 0;
    	for (i=0; i < NumberOfColors*4; ++i)
    	{
    	   tmpByte = INPUT[i] ^ TABLE[i];
    	   OUTPUT[i] = tmpByte ^ prevByte;
    	   prevByte = INPUT[i];
    	}

where:

`INPUT` and `OUTPUT` represent the encoded file bytes and the decoded output bytes `TABLE` represents a fixed set of values defined as:

    	2E 82 E4 6F  38 A9 DC C6  7B B6 28 AC  FD AA 8A 4E
    	76 2E F0 E4  25 1B 8A 68  4E 92 B9 B4  95 F0 3E EF
    	F7 40 24 18  39 31 BB E1  53 A8 1F B1  3A 07 FB CB
    	E6 00 81 50  0E 40 E1 2C  73 50 0D 91  D6 0A 5D D6
    	8B B8 62 AE  47 00 53 5A  B7 80 AA 28  F7 5D 70 5E
    	2C 0B 98 E3  A0 98 60 47  89 9B 82 FB  40 C9 B4 00
    	0E 68 6A 1E  09 85 C0 53  81 D1 98 89  AF E8 85 4F
    	E3 69 89 03  A1 2E 8F CF  ED 91 9F 58  1E D6 84 3C
    	09 27 BD F4  C3 90 C0 51  1B 2B 63 BC  B9 3D 40 4D
    	62 6F E0 8C  F5 5D 08 FD  3D 50 36 D7  C9 C9 43 E4
    	2D CB 95 B6  F4 0D EA C2  FD 66 3F 5E  BD 69 06 2A
    	03 19 47 2B  DF 38 EA 4F  80 49 95 B2  D6 F9 9A 75
    	F4 D8 9B 1D  B0 A4 69 DB  A9 21 79 6F  D8 DE 33 FE
    	9F 04 E5 9A  6B 9B 73 83  62 7C B9 66  76 F2 5B C9
    	5E FC 74 AA  6C F1 CD 93  CE E9 80 53  03 3B 97 4B
    	39 76 C2 C1  56 CB 70 FD  3B 3E 52 57  81 5D 56 8D
    	51 90 D4 76  D7 D5 16 02  6D F2 4D E1  0E 96 4F A1
    	3A A0 60 59  64 04 1A E4  67 B6 ED 3F  74 20 55 1F
    	FB 23 92 91  53 C8 65 AB  9D 51 D6 73  DE 01 B1 80
    	B7 C0 D6 80  1C 2E 3C 83  63 EE BC 33  25 E2 0E 7A
    	67 DE 3F 71  14 49 9C 92  93 0D 26 9A  0E DA ED 6F
    	A4 89 0C 1B  F0 A1 DF E1  9E 3C 04 78  E4 AB 6D FF
    	9C AF CA C7  88 17 9C E5  B7 33 6D DC  ED 8F 6C 18
    	1D 71 06 B1  C5 E2 CF 13  77 81 C5 B7  0A 14 0A 6B
    	40 26 A0 88  D1 62 6A B3  50 12 B9 9B  B5 83 9B 37


### Section 1 - Stitch Attributes (Same for HUS & VIP)

Section 1 defines the per-stitch attributes of the pattern. This section is generally much smaller than the other two, but that's only because the data in this section compresses more readily than in the others. The section is compressed using the proprietary ArchiveLib compression library made by [Greenleaf Software](http://www.greenleafsoft.com). This library supports several different compression schemes, but the one used in this case is called AL_GREENLEAF_LEVEL_4. Once decompressed, the data should be exactly NumberOfStitches bytes in length with each byte sequentially representing one stitch of the pattern.

| Attribute | Stitch type            |
| --------- | ---------------------- |
| `0x80`    | Normal stitch          |
| `0x81`    | Jump stitch            |
| `0x84`    | Color change           |
| `0x90`    | Last stitch in pattern |

### Sections 2 & 3 (Same for HUS & VIP)

These sections are compressed using exactly the same method as Section 1, but they define the X and Y components of the pattern. They each decompress to exactly NumberOfStitches bytes in length. The first byte from Section 2 defines the X-coordinate of the first stitch in the pattern. Likewise, the first byte of Section 3 defines the Y-coordinate. Together with the first byte of Section 1, we get the full definition of the first stitch. Subsequent bytes correspond to one another in the same way.

The byte values of the second and third sections are signed and represent the relative movement from the previous stitch. These offsets are measured in 0.1mm units. For example, if the first bytes of the sections was `0x81, 0x45, 0xC9`, this would be interpreted as a *Jump Stitch* moving `X+6.9mm, Y-5.5mm`.

### Acknowledgements

**Jimmy Engström** - For pointing out the existence of the VIP format and how closely it resembled HUS...and for reminding me of this old project lurking on my hard drive. :)

---

Copyright **Jason Weiler** 2006. All Rights Reserved.
