---
title: "rgbCTF - Vaporwave2"
categories:
  - rgbCTF
tags:
  - rgbCTF
---

*I got two versions. I got twooo versions…*

*~Quintec#0689*

---

For this challenge, we're given two files, vaporwave2a.flac and vaporwave2b.flag. Both files sound like the exact same song, so we'll use Audacity to find any hidden message:

First, we need to import one of the files into Audacity, select the entire sound area with CTRL+A, go to effects, click invert, and wait for the sound to be inverted. Once this is complete, we can import the other file, and play the sounds together. 

In short, playing a wave's inverse at the same time as a wave causes the sound to be inaudible. Because the two files sound the same, any parts of the files that sound exactly the same are no longer audible when we play the inverse of one file against the other, allowing us to hear any hidden messages.

Once we've imported the second file, we can export the entire sound to a .wav file and use a spectrum analyzer to grab the flag:

https://academo.org/demos/spectrum-analyzer/

**rgbCTF{s3v3r3_1nv3r71g0}**