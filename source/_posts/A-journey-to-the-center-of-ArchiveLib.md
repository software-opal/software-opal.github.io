---
title: A journey to the center of ArchiveLib
date: 2019-01-23 22:35:56
tags: 
 - Machine Embroidery
 - ArchiveLib
 - embroidery-rust
categories:
 - 
  - Coding
  - Machine Embroidery
---

So, it's been a long 3 days at LCA2019 and I'm thinking about doing a lightning talk on my latest project: Machine Embroidery. Specifically, the sizable amount of software acheology that's been involved in implementing parsing(that still isn't done) 2 specific embroidery formats `VIP` and `HUS`.

This is a rabbit hole; but one I am slowly backing up out of. So here we go.

## The formats

Both formats follow the [same structure](http://www.jasonweiler.com/HUSandVIPFileFormatInfo.html): a header, color information, and 3 sections of compressed stitch data. Let's take an example file and run through it. The file we're using stitches this:

![An embroidered Trans pride flag in the shape of the Star Wars Rebel Alliance logo](Rebel Pride.jpg)

So ... the header says we have a `HUS` file with 22972(`0x0000_59BC`) stitches and 6(`0x0000_0006`) colours(it's complicated; but each stripe is a different 'colour' even if it was stitched with the same colour thread #creativeLicense). Some other useful stuff for the machine like bounds; but we can skip that stuff. Finally the three offsets: `0x0000_0036`, `0x0000_00bc`, and `0x0000_09fe`.

```
00000000  5b af c8 00 bc 59 00 00  06 00 00 00 52 02 5a 02  |[....Y......R.Z.|
00000010  00 00 dc fe 36 00 00 00  bc 00 00 00 fe 09 00 00  |....6...........|
00000020  00 00 00 00 00 00 00 00  00 00                    |..........|
0000002a
```

For brevity; I'm going to skip the colour parsing because it is just convoluted. Seriously ... just write out RGB values. Please.

Now onto the three compressed block; Here is the first one which contains all the information about the stitch attributes(is it a jump stitch, regular, colour change, or the end of the file).

```hexdump
00000036  00 8c 53 4c a0 4a d7 fe  9b 1e 93 88 69 6f ff 56  |..SL.J......io.V|
00000046  69 a2 ec 2e 09 1d 3d 0d  a0 36 83 14 0e 39 91 09  |i.....=..6...9..|
00000056  a0 d4 46 80 b5 06 f9 59  00 47 23 12 37 36 6f 61  |..F....Y.G#.76oa|
00000066  83 2c dc a3 5a 3e 10 31  de 67 37 19 1c 30 f1 d5  |.,..Z>.1.g7..0..|
00000076  cd d9 ee fa 3a ef 00 00  00 00 ee 8b 2a 85 0e 69  |....:.......*..i|
00000086  97 a8 cf 87 a7 00 2c 88  aa 0c 00 d1 17 55 0c 00  |......,......U..|
00000096  00 00 00 00 00 68 8b 3c  50 13 3e 7b f0 00 00 00  |.....h.<P.>{....|
000000a6  3d 22 ea a1 83 54 45 50  7d 9f ac 63 da 66 5e 60  |="...TEP}..c.f^`|
000000b6  00 03 eb 6d df c0                                 |...m..|
000000bc
```

This is mostly garbage to me so onto this compression algorithm

## The compression

As described on the reference page this 138 byte block is compressed using a proprietary archive format called [Greenleaf ArchiveLib](https://web.archive.org/web/20130226125149/http://www.greenleafsoft.com/ArcLib/ArchiveLibSummary.asp). Not to be confused with `libarchive` which is a completely different kettle of fish.

This is a compression algorithm that a company could buy the source code for and compile into their applications with any C++ compiler. Judging by [this FAQ page archived by WayBack](https://web.archive.org/web/19991128070108/http://greenleafsoft.com) it has a number of benefits over some other algorithms of the age.

But, enough about the product; lets get to some code. Obviously the code exists on the internet; probably by accident. It's of a old release(version 1.0) released around 1994/1995 but it has the compression algorithm in all its glory:

```C++ Oh ... Oh https://github.com/software-opal/archivelib-rs/blob/82d224e48e00393107734aae1ecc364bd9695748/archivelib-sys/c-lib/src/_rc.cpp Source
/* ... snip ... */
#define _519 "Incorrect compression level parameter passed to compressor.  Compression level = %d"
#define _520 "Memory allocation failure in compression startup"
#define _445(_200,_446)((short )((_446<<_154)^(_278[_200+2]))&(_153-1))
#define _447(_200,_201){short _204;if((_204=_163[_201])!=_157)_164[_204]=\
_200;_164[_200]=_201;_163[_200]=_204;_163[_201]=_200;}
#define _448(s){short _204;if((_204=_164[s])!=_157){_164[s]=_157;_163[_204]\
=_157;}}
RCompress::RCompress(ALStorage&_266, ALStorage&_267, int _269, int _235)
{ _161=&_266; _162=&_267; _531=_235; if(_269>_137||_269<_138){
mStatus.SetError(AL_ILLEGAL_PARAMETER,_519,_269-10); _175=2; }else
_175=(short )(1<<_269); _176=(short )(_175-1); if((_166=new uchar
[_175+_140+2])!=0) memset(_166,0,(_175+_140+2)*sizeof(uchar ));
/* ... snip ... */
```

Did I say glory; I meant gory. This code is minified, obfuscated C++. Cool. It's source code. We can make this work.

Oh; side note, I want this embroidery library to be pure Rust, so whilst I could(and did for a time) link against this code, that is not 'good enough'.

### Step 1: Unminifying

So, the first step is to just get some resemblance of indentation; layout; something; anything; please.

```C++ It's better? https://github.com/software-opal/archivelib-rs/blob/434bbc47b18a09ad6662215f8ade45e1063f398b/archivelib-sys2/c-lib/src/_rc.cpp Source
/* ... snip ... */
#define _519                                                                   \
  "Incorrect compression level parameter passed to compressor.  Compression "  \
  "level = %d"
#define _520 "Memory allocation failure in compression startup"
#define _445(_200, _446)                                                       \
  ((short)((_446 << _154) ^ (_278[_200 + 2])) & (_153 - 1))
#define _447(_200, _201)                                                       \
  {                                                                            \
    short _204;                                                                \
    if ((_204 = _163[_201]) != _157)                                           \
      _164[_204] = _200;                                                       \
    _164[_200] = _201;                                                         \
    _163[_200] = _204;                                                         \
    _163[_201] = _200;                                                         \
  }
#define _448(s)                                                                \
  {                                                                            \
    short _204;                                                                \
    if ((_204 = _164[s]) != _157) {                                            \
      _164[s] = _157;                                                          \
      _163[_204] = _157;                                                       \
    }                                                                          \
  }
RCompress::RCompress(ALStorage &_266, ALStorage &_267, int _269, int _235) {
  _161 = &_266;
  _162 = &_267;
  _531 = _235;
  if (_269 > _137 || _269 < _138) {
    mStatus.SetError(AL_ILLEGAL_PARAMETER, _519, _269 - 10);
    _175 = 2;
  } else
    _175 = (short)(1 << _269);
  _176 = (short)(_175 - 1);
  if ((_166 = new uchar[_175 + _140 + 2]) != 0)
    memset(_166, 0, (_175 + _140 + 2) * sizeof(uchar));
/* ... snip ... */
```

Wow, that is so much not any better. The formatting means I can see that I'm looking at some `#define`s and a function; but besides that I don't know what `_531` is an what it's doing(spoiler: the `int` is a lie). Also, because this is all on a C++ class there is no easy differentiator between `_531` which is a attribute on `this` and `_137` which is a `#define`d constant and `_269` which is a parameter. 

### Step 2: Culling

There is a lot of associated code, quite a few `#ifdef`s and some weird DOS specific flags that were getting in the way; so now that I can see what I'm looking at, I started purging all of them. Huge swathes of code got deleted. I was aiming for the compression algorithm and the smallest amount of auxiliary code to support it.

At the same time I needed to standardize types; In C/C++ land `int` isn't an specific size; but in Rust everything is sized exactly, so to save myself some time in the future I changed all the types over; `char` became `uint8_t`, `short` became `int16_t` and so on.

### Step 3: Renaming

Finally; a small enough code base, that still compiles. Lets make it easier to identify the source of each of these variables: `#define`d variables become `CONST_xxx`; locals, arguments, functions and the items on the class all get their own prefix:

```C++ Getting better https://github.com/software-opal/archivelib-rs/blob/1453cebdbb0b183a6ba1187f5eb556675154a435/archivelib-sys2/c-lib/src/_rc.cpp Source
/* ... snip ... */
#define MACRO_FN445(arg278, arg200, arg446)                                    \
  ((int16_t)((arg446 << CONST__154) ^ (arg278[arg200 + 2])) & (CONST__153 - 1))
#define MACRO_FN447(arg163, arg164, arg200, arg201)                            \
  {                                                                            \
    int16_t macro_local204;                                                    \
    if ((macro_local204 = arg163[arg201]) != CONST__157)                       \
      arg164[macro_local204] = arg200;                                         \
    arg164[arg200] = arg201;                                                   \
    arg163[arg200] = macro_local204;                                           \
    arg163[arg201] = arg200;                                                   \
  \
#define MACRO_FN448(arg163, arg164, s)                                         \
  {                                                                            \
    int16_t macro_local204;                                                    \
    if ((macro_local204 = arg164[s]) != CONST__157) {                          \
      arg164[s] = CONST__157;                                                  \
      arg163[macro_local204] = CONST__157;                                     \
    }                                                                          \
  }
RCompress::RCompress(ALStorage &_266, ALStorage &_267, int32_t _269,
                     int32_t _235) {
  _161 = &_266;
  _162 = &_267;
  _531 = _235;
  if (_269 > CONST__137 || _269 < CONST__138) {
    mStatus.SetError(AL_ILLEGAL_PARAMETER, _519, _269 - 10);
    _175 = 2;
  } else
    _175 = (int16_t)(1 << _269);
  _176 = (int16_t)(_175 - 1);
  if ((_166 = new uint8_t[_175 + CONST__140 + 2]) != 0)
    memset(_166, 0, (_175 + CONST__140 + 2) * sizeof(uint8_t));
/* ... snip ... */
```

### Step 4: A struct for the data

With my eyes firmly on the Rust prize and a recollection of earlier attempts I knew that a C++ class wasn't going to translate well ... so let's move all that data into a `struct` and just re-work some of the allocation. Simple find and replace later and we have a data struct and a basic class to hold onto it.

### Step 5: Splitting the problem

At this point much of the auxiliary code was gone and I was left with 8 C++ files; mostly manageable; but the compress file was over 500 lines long at this point and I have this aversion to long files(I tend to get lost easily in them).

Woo. Now I have lots of files with code I don't understand instead of one big one ... wait ... how is that better?!

### Step 6: Refactoring

Not much to say ... I spent a lot of time trying to come up with names for variables that would fit in well with all the references of it; and it was mostly unsuscessfull; but I managed to identify several 'structures' and gain enough of an understanding of the algorithm to describe it:

### Step 7: Finally, Rust

Overall this was less painful than I had anticipated at the start(and from previous attempts). The work I'd done, especially in the refactoring, had meant I'd gained an understanding of the algorithm so I could un-`unsafe` it(is that safeify?).

## Conclusion

This algorithm is now written purely in Rust; with tests that use the old un-modified C++ version to verify the correctness. I have also take the opportunity to rewrite the expansion algorithm in more idiomatic Rust; the Compression algorithm though eludes my understanding.

You can find the code at [archive-rs](https://github.com/software-opal/archivelib-rs) and it will be on [Cargo](https://crates.io/crates/archivelib) once I clear up some licensing questions.

So how does the compression algorithm compress?

{% blockquote %}
ArchiveLib generates lookup tables; and then looks up either the byte itself, or the number of bytes to look back in the output to copy from.
{% endblockquote %}

And how do I feel about Rust?

{% blockquote Rick Astley %}
Never going to give you up
{% endblockquote %}
