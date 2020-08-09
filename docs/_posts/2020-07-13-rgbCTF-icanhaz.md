---
title: "rgbCTF: icanhaz"
categories:
  - rgbCTF
tags:
  - rgbCTF
---

*can u haz a meaningful career in cybersecurity tool development? we hope so!*

*~tonyz#2403*


For this challenge, we're given a file with a hex dump in it:

*Shortened for brevity:*
```
00000000: FD37 7A58 5A00 0004  ..:.!...
00000008: E6D6 B446 0200 2101  WO......
...
000008d8: 5A4E 7002 B1C4 67FB  !+...D..
000008e0: 0200 0000 0004 595A  .......!
```

We can parse through this hex dump to get useable hex values, that we can convert back to the original form of the file:

*Shortened again:*
```
$ cat icanhaz | cut -d : -f 2 | cut -c-20 | tr -d ' ' | tr -d '\n' | fold -w2 | paste -sd'\\' | fold -w3 | paste -sd'x'
FD\x37\x7A\x58\x5A\x00\x00\x04\xE6\xD6\xB4\x46\x02\x00
...
A2\x61\xA7\xE4\x2B\x00\x01\xC4\x11\x9E\xD8\x08\x00\x5A\x4E\x70\x02\xB1\xC4\x67\xFB\x02\x00\x00\x00\x00\x04\x59\x5A
```

Now that we have the original hex, we can use Python to print it to a file:

```
$ python -c 'print "\xFD\x37\x7A\x58\x5A\x00\x00\x04\xE6\xD6\xB4\x46\x02\x00..."' > output
...
$ file output
out: XZ compressed data
```

It looks like we have a compressed file, so we'll rename it to output.xz and decompress it with 7z:

```
$ 7z x output.xz
...
output: SVG Scalable Vector Graphics image
```

Taking a look at the image shows us what appears to be a blank image:

![blank](https://i.ibb.co/R62fRbL/cap0.png)

We can use StegOnline to see if there's anything interesting in the image:

https://georgeom.net/StegOnline/image

Checking the "LSB Half" option shows us a very faint QR code:

![faint](https://i.ibb.co/8d77bHm/cap1.png)

If we increase the saturation of the image, replace all white with blank, then invert the image, we get a useable QR code:

![QRCode3](https://i.ibb.co/qJcTZdM/cap2.png)

Now, we can use an online QR Code reader to read the image:

https://zxing.org/w/decode

We're given another long string that, when base64 decoded redirected to a file, gives us another xz file. If we unzip the new xz file using 7z as we did above, we get a final QR code, containing the flag:

```
█████████████████████████████████
█████████████████████████████████
████ ▄▄▄▄▄ █▀▀ ███ ▀▀█ ▄▄▄▄▄ ████
████ █   █ █▄▀██▀▀▀ ▀█ █   █ ████
████ █▄▄▄█ █ ▄ █  ▄ ██ █▄▄▄█ ████
████▄▄▄▄▄▄▄█ █ ▀▄█ █▄█▄▄▄▄▄▄▄████
████ ▄▀ ▀▀▄▄▀▀█  ▀   ▀ ▄▄▀▄ ▀████
████▄█▀▄▀▀▄█ ▀▀  ▀▀▀▀▀▄▀▀█▄  ████
████▄ █▀ █▄ ██▄ █▀██▀  ▀▄▀   ████
████▄▀█▄█▄▄ ▄▀█ █ ██▄▀▀ ▀▄█▀ ████
████▄▄▄▄██▄▄▀▀ █ ▄▀▄ ▄▄▄ ▄█▀ ████
████ ▄▄▄▄▄ █▄█▀▄ ▄▀▄ █▄█ █  ▄████
████ █   █ █▀█▄▀▄▀▄█▄▄▄  █▄▄█████
████ █▄▄▄█ █▀▄██▀▀ ▀▀█ █▄█▄█▄████
████▄▄▄▄▄▄▄█▄▄█▄█▄█▄▄█▄██▄█▄▄████
█████████████████████████████████
█████████████████████████████████
```

**rgbCTF{iCanHaz4N6DEVJOB}**
