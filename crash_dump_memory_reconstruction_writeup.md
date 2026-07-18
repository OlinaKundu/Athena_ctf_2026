# Crash Dump Memory Reconstruction Writeup

## Challenge Description

A crash dump from an analyst workstation was captured before the system
was wiped. Plain string extraction revealed only small fragments of
useful information, suggesting that the interesting data was fragmented
across multiple memory pages. Along with the memory snapshot, a process
map was provided. The objective was to identify the correct process,
reconstruct the resident buffer, reverse the page-level transformation,
and recover the hidden flag.

------------------------------------------------------------------------

# Solution

## Initial Analysis

The challenge provided two files:

-   `process_map.txt`
-   `ram_dump.txt`

I began by examining the process map to determine which process owned
the dumped memory pages.

``` text
pid  name               base      size    xor tag
111  systemd            0x7f1000  0x1200  tag=STEDY
482  explorer           0x7f4000  0x2400  tag=CLIP9
901  photo_recover      0x8a0000  0x3c00  tag=DR1FT
1044 crash_handler      0x900000  0x1800  tag=PGOFF
```

The memory pages (`0x8a0000`, `0x8a1000`, and `0x8a2000`) all belonged
to the `photo_recover` process, whose repeating XOR key was `DR1FT`.

## Inspecting the Memory Dump

The first two pages contained the recognizable forensic sentinel values:

``` text
DE AD BE EF
FA CE B0 0C
```

I treated these markers as dump artifacts rather than resident data.
Everything following each marker was ignored.

The reconstructed resident buffer therefore consisted of:

### Page 0x8a0000

``` text
25 26 59 23 3a 25 29 43 27 39 1b 22
```

### Page 0x8a1000

``` text
50 21 31 37 0d 59 2f 30 21 0d 57 34
```

### Page 0x8a2000

``` text
35 23 3f 54
28 20 37 2f
0c 1a 05 07
0c 0e 07 0f
06 1d 10 12
0a 0b 0d 1f
```

Concatenating these fragments produced the complete 48-byte resident
buffer.

## Undoing the XOR

The process-specific key was:

``` text
DR1FT
```

Applying the repeating XOR across the reconstructed buffer decoded the
plaintext:

``` text
athena{ram_pages_hide_fragments}
```

The remaining decoded bytes after the closing brace were unrelated
memory, confirming the end of the flag.

# Flag

``` text
athena{ram_pages_hide_fragments}
```

# Conclusion

This challenge demonstrated how fragmented memory pages can be
reconstructed using process metadata before reversing a simple
page-level XOR transformation. By identifying the correct process,
removing forensic sentinel artifacts, rebuilding the contiguous buffer,
and applying the correct XOR key, I successfully recovered the hidden
flag.

**Flag:** `athena{ram_pages_hide_fragments}`
