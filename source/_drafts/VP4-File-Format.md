---
title: VP4 File Format
tags:
---

In the absence of any information in the great fountian of knowledge that is the internet I am going to reverse engineer the `VP4` file format.

`VP4` is a file format designed for PFAFF embroidery machines([source](https://forum.embroideres.com/announcement/3-vp4-pfaff-embroidery-format/)). It appears both in name and content to be based off the previous VP3 file format that was documented by [Jason Weiler](http://www.jasonweiler.com/VP3FileFormatInfo.html)(also avaliable ...). 

For this investigation I will be using [this lovely pattern](https://forum.embroideres.com/files/file/2432-blue-flower-and-green-leaf-free-embroidery-design/) which is avaliable for free in a wide variety of formats(thank you 💖), and for I'll specifically be making use of the [DST](blueflowle.dst), [VP3](blueflowle.vp3) and [VP4](blueflowle.vp4) formats.  The picture below shows the pattern in all it's glory.

![An embroidered blue flower with green leaves on a plain black background](flower-realistic.jpg)

## Getting stuck in


Lets have a look at the top portion of the VP4 file. The file is `0x1e98` bytes long

```
00000000  25 56 70 34 25 01 00 00  00 4d 73 61 0a df 74 29  |%Vp4%....Msa..t)|
00000010  3c 87 6b 44 2c 84 2f 00  3c 7c f7 e7 a0 69 6e 66  |<.kD,./.<|...inf|
00000020  6f 00 00 00 00 2c 00 00  00 6e 74 74 6e 00 00 00  |o....,...nttn...|
00000030  00 0e 00 00 00 02 00 6e  74 65 73 00 00 73 74 67  |.......ntes..stg|
00000040  73 00 00 5f 0e 00 00 04  00 00 00 01 00 29 ff 1a  |s.._.........)..|
00000050  fe d6 00 e5 01 01 68 6f  6f 70 00 00 00 00 2c 00  |......hoop....,.|
00000060  00 00 0b 00 50 66 66 5f  32 30 30 78 32 30 30 18  |....Pff_200x200.|
00000070  00 50 66 61 66 66 20 63  72 65 61 74 69 76 65 20  |.Pfaff creative |
00000080  73 65 6e 73 61 74 69 6f  6e 01 d0 07 d0 07 00 73  |sensation......s|
00000090  62 64 73 00 00 00 00 fd  1d 00 00 01 00 73 62 64  |bds..........sbd|
000000a0  6e 00 00 00 00 ef 1d 00  00 00 00 00 00 00 00 00  |n...............|
000000b0  00 00 00 00 00 00 00 00  00 00 6e 74 74 6e 00 00  |..........nttn..|
000000c0  00 00 0e 00 00 00 02 00  6e 74 65 73 00 00 73 74  |........ntes..st|
000000d0  67 73 00 00 00 04 00 00  00 a8 ff 48 ff 74 68 72  |gs.........H.thr|
000000e0  64 00 00 00 00 2b 00 00  00 10 00 4d 61 64 65 69  |d....+.....Madei|
000000f0  72 61 20 52 61 79 6f 6e  20 34 30 04 00 31 30 39  |ra Rayon 40..109|
00000100  32 0a 00 43 61 64 65 74  20 42 6c 75 65 08 28 8f  |2..Cadet Blue.(.|
00000110  d2 e9 00 00 8a 07 00 00  00 00 00 00 00 00 f2 eb  |................|
00000120  f7 e9 fa e8 fa e8 f8 e9  f7 e9 f5 ed f2 f4 f0 fa  |................|
00000130  ee 01 11 f8 0d 00 15 07  12 0c 0f 0f 0f 10 0b f8  |................|
00000140  fa 0d f2 f1 f2 f1 f0 f3  ed f7 ef fc f4 03 f4 06  |................|
00000150  12 ff 10 06 0e 0b 0a 13  08 15 08 15 06 15 05 16  |................|
```

Here we can see some striking similarities with the VP3 format; an initial `%Vp4%` as opposed to `%Vsm%`
