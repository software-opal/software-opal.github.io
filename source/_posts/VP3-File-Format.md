---
title: VP3 File Format
date: 2019-01-28 16:48:38
categories:
  - Embroidery formats

---

[Source](https://www.jasonweiler.com/VP3FileFormatInfo.html "Permalink to VP3 Embroidery File Format")

## File Header Information

| Offset | Size | Description |
| --- | --- | :--- |
| 0x00 | 6 bytes | Magic Number: Always "%vsm%" + \0 |
| 0x06 | 2 bytes | (N1) Number of bytes in Identifier String |
| 0x08 | N1 bytes | Body of identifier string. Uses wide characters and may not be NULL terminated |
| 0x08+N1 | 2 bytes | Unknown |
| 0x09+N1 | 1 byte | Unknown |
| 0x0A+N1 | 4 bytes | Outer Section Size - Describes the number of bytes left in the file |
| 0x0C+N1 | 2 bytes | (N2) Number of bytes in unknown string |
| 0x0E+N1 | N2 bytes | Body of unknown byte string. |

## Hoop Configuration

| Offset | Size | Description |
| --- | --- | :--- |
| Offset Hoop Dimensions |
| 0x00 | 4 bytes | Positive X Hoop dimension in 1000ths of a millimeter |
| 0x04 | 4 bytes | Positive Y Hoop dimension in 1000ths of a millimeter |
| 0x08 | 4 bytes | Negative X Hoop dimension in 1000ths of a millimeter |
| 0x0C | 4 bytes | Negative Y Hoop dimension in 1000ths of a millimeter |
| 0x10 | 4 bytes | Unknown DWORD |
| 0x14 | 4 bytes | Unknown DWORD |
| 0x18 | 4 bytes | Unknown DWORD |
| 0x1C | 4 bytes | Number of bytes remaining in the file |
| 0x20 | 4 bytes | Origin X-Offset |
| 0x24 | 4 bytes | Origin Y-Offset |
| 0x28 | 1 byte | Unknown BYTE |
| 0x29 | 1 byte | Unknown BYTE |
| 0x30 | 1 byte | Unknown BYTE |
| Centered Hoop Dimensions |
| 0x31 | 4 bytes | Positive X Hoop dimension in 1000ths of a millimeter |
| 0x35 | 4 bytes | Negative X Hoop dimension in 1000ths of a millimeter |
| 0x39 | 4 bytes | Positive Y Hoop dimension in 1000ths of a millimeter |
| 0x3D | 4 bytes | Negative Y Hoop dimension in 1000ths of a millimeter |
| 0x41 | 4 bytes | Hoop width in 1000ths of a millimeter |
| 0x45 | 4 bytes | Hoop height in 1000ths of a millimeter |
| More Unknowns... |
| 0x47 | 2 bytes | Unknown WORD |
| 0x49 | 1 byte | Unknown BYTE |
| 0x4A | 1 byte | Unknown BYTE |
| 0x4D | 1 byte | Unknown DWORD |
| 0x52 | 1 byte | Unknown DWORD |
| 0x46 | 1 byte | Unknown DWORD |
| 0x4A | 1 byte | Unknown DWORD |

## Stitch Section Header

| Offset | Size | Description |
| --- | --- | :--- |
| 0x00 | 6 bytes | Magic Number: Always 0x78 0x78 0x55 0x55 0x01 0x00 |
| 0x06 | 2 bytes | (N3) Number of bytes in Identifier String (same as above) |
| 0x08 | N3 bytes | Body of identifier string. Uses wide characters and may not be NULL terminated (same as above) |
| 0x0A+N3 | 2 bytes | Number of colors in the pattern |

## Color Section

Each color is composed of a single color section header followed by a single color section body.

### Color Section Header

| Offset | Size | Description |
| --- | --- | :--- |
| 0x00 | 3 bytes | Unknown Bytes |
| 0x03 | 4 bytes | Offset to next color section as an offset from the top of the color section. |
| 0x07 | 4 bytes | Unknown X-Offset in 1000ths of a millimeter |
| 0x0B | 4 bytes | Unknown Y-Offset in 1000ths of a millimeter |
| 0x0F | 1 byte | (TS) Unknown table size multiplier |
| 0x10 | 1 byte | Thread color blue channel |
| 0x11 | 1 byte | Thread color green channel |
| 0x12 | 1 byte | Thread color red channel |
| 0x13 | 6*TS | Unknown table |
| Color descriptions strings |
| 0x00 | 2 bytes | (S1) Size of color string 1 |
| 0x02 | S1 bytes | Color string 1 - ascii, no NULL terminator |
| 0x02+S1 | 2 bytes | (S2) Size of color string 2 |
| 0x04+S1 | S2 bytes | Color string 2 |
| 0x04+S1+S2 | 2 bytes | (S3) Size of color string 3 |
| 0x06+S1+S2 | S3 bytes | Color string 3 |
| Color descriptions strings |
| 0x00 | 4 bytes | Unknown X-Offset in 1000ths of a millimeter |
| 0x04 | 4 bytes | Unknown Y-Offset in 1000ths of a millimeter |
| 0x08 | 2 bytes | (N4) Number of bytes in unknown string |
| 0x0A | N4 bytes | Body of unknown byte string. |
| 0x0A+N4 | 4 bytes | (NS) Number of stitch bytes for this color only. |
| 0x0E+N4 | 3 byte | Unknown BYTES |

### Color Section Body

The number of stitches can be approximated by subtracting 3 from
the number of stitch bytes (SB) above, and the dividing by 2.
Unfortunately, this is only a approximation of the number of stitches in
 this color. As we'll see, the actual number of stitches can be lower
than this calculation due to special stitches.

From here to the end of the section, each regular stitch is
represented as two sequential bytes. The first byte represents the
X-component of the stitch as a signed integer in 10th of a millimeter,
and the second byte represents the Y-component.

## Special Stitches

When either the X component of the stitch has the value of 0x80
then we have a special stitch. The Y-component of the special stitch
tells us how to handle it. When Y is 0x00 or 0x03, the special stitch is
empty and the two bytes already read are ignored. When the Y component
is 0x01, then the next 6 bytes represent a jump stitch. The first 2
bytes is the X-component of the jump stitch as a signed word in 10ths of
a millimeter. As expected, the subsequent 2 bytes is the Y-component of
the jump stitch. The final 2 bytes is a stitch terminator and has
consistently been 0x80 0x02.

Each stitch is an offset from the previous one. Once all bytes
from a given color are parsed, the next color section picks up in the
same position where the current one left off.

---

Copyright **Jason Weiler** 2006. All Rights Reserved.
