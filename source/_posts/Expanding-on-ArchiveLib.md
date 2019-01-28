---
title: Expanding on ArchiveLib
date: 2019-01-25 21:43:54
tags:
 - ArchiveLib
 - embroidery-rust
categories:
 - 
  - Coding
---

## Background

ArchiveLib is a proprietary algorithm for compressing code. I wrote about the process of turning the code from obfuscated C into rust here: {% post_link A-journey-to-the-center-of-ArchiveLib %}. In this post I want to provide a psuedo-implementation of the algorithm in plain-ish language so that others can implement it.

The easier of the two algorithms to reason about is the Expansion/Decompression algorithm; so this is described here.


### Some conventions

A lot of the work to read this data relies on bitwise operations; and worse reading weird numbers of bits from weird offsets; so some conventions:

 - Bits within a byte are represented as `byte(hex)/bit`; so to read the `F` from the 2nd byte: `01/4-01/8`.
 - Big-endian everywhere(largest byte on the left).
 - All binary numbers will have underscores where the 4-bit boundry exists in the file. For example, `01/2-01/8` is `0b11_1111`
 - All numbers are decimal, except the byte offsets; and where one of `0b`, `0o` and `0x` which represent binary, octal and hexadecimal respectively. For example `11 == 0xB == 0o13 == 0b1011`.
 - The code is python-ish; and the functions `read_unsigned` and `read_signed` takes two parameters: `start_bit` and `bit_count` and return either a signed or unsigned integer.
 
### Some caveats

I have learnt this by spending many hours staring at code in various forms of obfuscation, and whilst I hope that this guide will be clear I expect there to be things that are not. If you find things like that; please tweet/message me on Twitter or open an issue on GitHub.

### Some helpers

Some helpers that help clarify code

#### read_coded_length

In quite a few places this idea of extending the size of a value exists. To give an idea of what it's doing; take this table:

| Bit pattern | Read value |
| ----------- | ---------- |
| `0b000`     | 0          |
| `0b001`     | 1          |
| ...         | ...        |
| `0b110`     | 6          |
| `0b1110`    | 7          |
| `0b11110`   | 8          |
| `0b111110`  | 9          |
| `0b1111110` | 10          |

```python
def read_coded_length(start_bit, initial_length):
  length = read_unsigned(start_bit, initial_length)
  start_bit += initial_length
  if (length + 1) == (1 << initial_length):
    while read_unsigned(start_bit, 1) == 1:
      length += 1
      start_bit += 1
    start_bit += 1  # Accounting for the final zero bit.
  return (start_bit, length)
```

## In the beginning

We need some sample data to begin with. To take advantage of some of the strengths of this library; repetition is heavily used.

```text Input
The code is easy.
The code is hard.
Some of the code has bugs.
But the code is not special.
```

```text Input(hexdump)
00000000  54 68 65 20 63 6F 64 65  20 69 73 20 65 61 73 79  |The code is easy|
00000010  2E 0A 54 68 65 20 63 6F  64 65 20 69 73 20 68 61  |..The code is ha|
00000020  72 64 2E 0A 53 6F 6D 65  20 6F 66 20 74 68 65 20  |rd..Some of the |
00000030  63 6F 64 65 20 68 61 73  20 62 75 67 73 2E 0A 42  |code has bugs..B|
00000040  75 74 20 74 68 65 20 63  6F 64 65 20 69 73 20 6E  |ut the code is n|
00000050  6F 74 20 73 70 65 63 69  61 6C 2E                 |ot special.|
0000005B
```

This produces the following output when compressed:

```text Output(hexdump)
00000000  00 3F 4B 52 8D AF F8 FB  80 3E 55 87 A6 A9 35 DA  |.?KR.....>U...5.|
00000010  AD D2 2B 58 DB 80 78 E4  82 C6 3C E0 C7 40 00 56  |..+X..x...<..@.V|
00000020  4A 82 6D 44 15 70 86 FD  94 BF 0A 9F 54 29 62 DB  |J.mD.p......T)b.|
00000030  A0 6D 04 7A 95 37 19 DB  57 29 61 68 F8 DE 70 E1  |.m.z.7..W)ah..p.|
00000040  A0 3F 29 3A 9E C5 78                              |.?):..x|
00000047
```

## Building our tables

The algorithm has 2 parts; the first(and harder to explain) part in the generation of a series of lookup tables. But before we get there we must first read the number of operations in this lookup table 

```python
operations = read_unsigned(0, 16);  # In our example this is 63.
```

### Our first tables: `tree_generator` & `tree_generator_len`

 - Bytes used `2/0-5/2`: `0b0100_1011_0101_0010_1000_1101_10`

So, now we have got that out of the way we can start building the first of our 3 tables.

```python
fill_count = read_unsigned(16, 5);  # In our example this is 9.
if fill_count == 0:  # Included for completeness.
  let tmp = read_unsigned(21, 5)
  let tree_generator = [tmp] * 256
  let tree_generator_len = [0] * 19
  return (tree_generator, tree_generator_len)
```

Now we can start reading out bytes into `tree_generator_len`. This sample will use `2/5` to `5/2`(`0b011_0101_0010_1000_1101_10`) which is :

```rust
offset = 21;
tree_generator_len = [0] * 19;
i = 0;
while i < fill_count:
  (offset, tree_generator_len[i]) = read_coded_length(offset, 3)
  i += 1
  if i == 3;
    i += read(offset, 2); // In our example, this is 2 at 3/6-3/8
    offset += 2;
```

In our example this produces the following items; with the remaining elements being zero: `[3, 2, 4, 0, 0, 4, 3, 3, 2, ...]`

### Tree generation

As a bit of an aside, there is a data structure that I describe as a tree; but if someone else has a better name then feel free to send it to me.

Let's take these two arrays:

```python
left = [0, 2, 3, 1]
right
```
